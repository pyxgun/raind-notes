---
layout: default
title: Path Traversal
permalink: /docs/cve_poc
toc_h2_only: true
---

ここでは、過去のCVEと同じ攻撃面が存在するかどうかの検証(PoC)を行い、その結果をまとめています。   
> [注意] 実際の攻撃手法は記載しておらず、あくまで攻撃が成功する可能性があるかどうかの確認までですが、ここに記載の手法を実環境等で実践しないようにご注意ください。記載された内容を実行したことで生じた問題の一切に責任を負いません。

**目次**
* TOC
{:toc levels="2..2"}

---

## CVE-2024-21626
### 概要
runc FDリーク系の脆弱性。  
runc内部で保持していたホスト側ディレクトリFDがコンテナプロセスに露出し、`/proc/self/fd`経由でホストFSに到達できるbreakoutが発生。[CVE詳細](https://access.redhat.com/security/vulnerabilities/RHSB-2024-001?utm_source=chatgpt.com)

### 確認手順
コンテナ内から`/proc/self/fd/*`に**ホスト側の実ファイルパス**が露出しないことを確認する。  

1. コンテナ内で以下を実行:
```bash
# 0/1/2以外のfdのリンク先を列挙（観測のみ）
for f in /proc/self/fd/*; do
  n=$(basename "$f"); [ "$n" -lt 3 ] && continue
  tgt=$(readlink "$f" 2>/dev/null || true)
  echo "$n -> $tgt"
done
```

2. 加えて、"危険度の高いパス"の露出有無のみ確認:
```bash
for f in /proc/self/fd/*; do
  tgt=$(readlink "$f" 2>/dev/null || true)
  echo "$tgt"
done | egrep '(^/etc/raind/|^/var/lib/|^/run/|^/sys/fs/cgroup|^/proc/1/root|^/root)'
```
期待値は以下の通り。

- fd:0/1/2以外は socket:[...], anon_inode:[...], /dev/null 等
- ホストの実パス(/etc/...,/sys/fs/...など)が出ない

### PoC
Raindでの実行結果は以下の通り。
```bash
$ raind container run --rm -t ubuntu
root@01kffwvmmrt1:/# 
root@01kffwvmmrt1:/# for f in /proc/self/fd/*; do
  n=$(basename "$f"); [ "$n" -lt 3 ] && continue
  tgt=$(readlink "$f" 2>/dev/null || true)
  echo "$n -> $tgt"
done
255 -> 
3 -> 

root@01kffwvmmrt1:/# for f in /proc/self/fd/*; do
  tgt=$(readlink "$f" 2>/dev/null || true)
  echo "$tgt"
done | egrep '(^/etc/raind/|^/var/lib/|^/run/|^/sys/fs/cgroup|^/proc/1/root|^/root)'
root@01kffwvmmrt1:/#
```
期待値通りの結果となった。

### 関連対策
本脆弱性に関連する対策は以下の通り。

- [FDリークと対策]({{ '/_posts/fd_leak' | relative_url }})

---

## CVE-2019-5736
### 概要
任意コード実行(ACE)系の脆弱性。  
コンテナ内rootがruncの実装上の欠陥を利用して **ホスト上のruncバイナリを上書き**し、その後ホストでruncが実行された際に任意コード実行につながった。[CVE詳細](https://nvd.nist.gov/vuln/detail/cve-2019-5736?utm_source=chatgpt.com)

### 確認手順
Raindにおけるrunc相当のコンポーネント、Dropletの実行ファイルや管理領域が、コンテナから「書き換え可能な形で到達」できないことを確認する。  

1. コンテナ内で以下を確認:
```bash
# runtimeディレクトリ /etc/raind が見えるか（存在チェックだけ）
ls -ld /etc/raind 2>/dev/null || true
```
2. コンテナに対しホスト上のバイナリファイル関連のディレクトリ(/bin,/usr/bin,/usr/local/bin)をマウントできないことを確認
3. 実行ファイルをマウント可能なディレクトリに移しマウントを行った場合に、実行不可であることを確認

### PoC
Raindでの実行結果は以下の通り。
```bash
# ホスト上のruntimeディレクトリと同一のパスの存在確認
root@01kffy77h400:/# ls -ld /etc/raind 2>/dev/null || true
root@01kffy77h400:/#
```
ホスト上のruntimeディレクトリが見えないことを確認。

```bash
# /usr/binをマウントソースとして指定し起動
$ raind container run -t -v /usr/bin:/hostbin ubuntu
2026/01/21 18:31:45 websocket: close 1013: target not ready

# audit logにて起動失敗原因の確認
$ cat /etc/raind/log/audit.log | grep 01kffyagd515 | grep prepare | grep fail | jq .
{
  "ts": "2026-01-21T18:31:42.654217431+09:00",
  "log_version": "0.1.0",
  "event": "init",
  "runtime": "droplet",
  "result": "fail",
    :
  "error": {
    "stage": "prepare",
    "message": "invalid mount source: /usr/bin"
  }
}
```
/usr/binがinvalid mount sourceとして拒否されていることを確認。

```bash
# バイナリをtmpにコピー
$ cp /usr/bin/droplet /tmp

# コピーしたバイナリをマウントし起動
$ raind container run -t -v /tmp/droplet:/mnt_droplet ubuntu

# マウントオプション確認
root@01kffyqaf1wd:/# findmnt -o TARGET,OPTIONS | grep /mnt_droplet
└─/mnt_droplet          rw,nosuid,nodev,noexec,relatime

# ファイル属性確認
root@01kffyqaf1wd:/# ls -l mnt_droplet 
-rwxr-xr-x 1 root root 7583959 Jan 21 09:38 mnt_droplet

# 実行
root@01kffyqaf1wd:/# ./mnt_droplet
bash: ./mnt_droplet: Permission denied
```

```bash
# tmpごとマウントし起動
$ raind container run -t -v /tmp:/host_tmp ubuntu

# マウントオプション確認
root@01kffyx2n386:/# findmnt -o TARGET,OPTIONS | grep host_tmp
└─/host_tmp             rw,nosuid,nodev,noexec,relatime

# tmp内に実行ファイルがあることを確認
root@01kffyx2n386:/# ls /host_tmp | grep droplet
droplet

# 実行
root@01kffyx2n386:/# /host_tmp/droplet
bash: /host_tmp/droplet: Permission denied
```

ユーザ指定のマウントに対し"noexec"を含めたオプションでマウントができていること、バイナリ直接のマウント/ディレクトリ配下に設置してのマウントいずれに対しても、バイナリ実行ができないことを確認。

### 関連対策
本脆弱性に関連する対策は以下の通り。  
※掲載予定

---


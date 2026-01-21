---
layout: default
title: Home
---

# Contents
- [はじめに]({{ '/docs/introduction/' | relative_url }})
- [Raindコンテナランタイムスタック概要]({{ '/_posts/project_architecture' | relative_url }})

## ライフサイクル
- [コンテナ起動シーケンス#1 create+initモデル]({{ '/_posts/container_startup_sequence_1' | relative_url }})
- [コンテナ起動シーケンス#2 create+shim+initモデル(PTY接続)]({{ '/_posts/container_startup_sequence_2' | relative_url }})

## セキュリティ
### OCI Configuration
- [OCI Configurationに対するTOCTOU攻撃と対策]({{ '/_posts/oci_configuration_toctou' | relative_url }})

### Filesystem/Mount
- [FDリークと対策]({{ '/_posts/fd_leak' | relative_url }})
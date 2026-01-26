---
layout: default
title: Home
---

# Contents
- [はじめに]({{ '/docs/introduction/' | relative_url }})
- [Raindコンテナランタイムスタック概要]({{ '/_posts/project_architecture' | relative_url }})

---

## Lifecycle
- [コンテナ起動シーケンス#1 create+initモデル]({{ '/_posts/container_startup_sequence_1' | relative_url }})
- [コンテナ起動シーケンス#2 create+shim+initモデル(PTY接続)]({{ '/_posts/container_startup_sequence_2' | relative_url }})

## Security
### - AuthN/AuthZ
- [mTLSよる認証]({{ '/_posts/mtls' | relative_url }})
- [SPIFFEによる認可]({{ '/_posts/authz_spiffe' | relative_url }})

### - OCI Configuration
- [TOCTOU攻撃と対策]({{ '/_posts/oci_configuration_toctou' | relative_url }})

### - Filesystem/Mount
- [FDリークと対策]({{ '/_posts/fd_leak' | relative_url }})
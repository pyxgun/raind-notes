---
layout: default
title: Project Arcihtecture
---

# Raindコンテナランタイムスタック
Raindは3つのコンポーネントで構成されたコンテナランタイムスタックの総称です。  

- **Raind CLI**: Raindコンテナランタイムスタックの操作を提供するCLIツール
- **Condenser**: コンテナライフサイクル管理/イメージ管理を行う高レベルコンテナランタイム
- **Droplet**: コンテナの作成/起動/停止を行う低レベルコンテナランタイム

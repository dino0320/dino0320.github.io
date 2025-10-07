---
layout: my-post
title: "WSLのデフォルトユーザーを設定する"
date: 2025-10-07 00:00:00 +0000
categories: platform windows
page_name: configure-wsl-default-user
lang: ja
---

WSLのデフォルトユーザーを設定する方法を説明します。

## 環境
- Windows 11
- Ubuntu 24.04.3 LTS (WSL2 ディストリビューション)

WSLのディストリビューションに接続し、次のように `/etc/wsl.conf` ファイルを編集します。

```conf
[user]
default=<任意のユーザー名>
```

WSLを再起動します。これでデフォルトユーザーが更新されているはずです。

```bash
wsl --shutdown
wsl --distribution Ubuntu   # 更新したデフォルトユーザーとして接続される
```
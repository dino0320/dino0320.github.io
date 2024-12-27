---
layout: my-post
title: "WSLのUser IDについて"
date: 2024-12-27 00:00:00 +0000
categories: platform windows
page_name: about-user-id-of-wsl
lang: ja
---

WSLのUser ID(uid)について調べました。

## 参考ページ
- [Advanced settings configuration in WSL](https://learn.microsoft.com/en-us/windows/wsl/wsl-config)

## 環境
- Windows 10 64ビット
- Ubuntu 22.04.3 LTS (WSL2ディストリビューション)

## WSLのUser ID
WSLのデフォルトユーザーのUser ID(uid)とGroup ID(gid)はともに `1000` です。  
Ubuntu上で以下のように確認できます。(WSLデフォルトユーザー名がusernameのとき)  
```bash
cat /etc/passwd
～略～
username:x:1000:1000:,,,:/home/username:/bin/bash
```

## この記事を書くに至った経緯
LaravelアプリケーションをWSL上のDockerで開発中、`php artisan make:～略～` で作ったファイルがWSL上で実行中のVSCodeで編集できませんでした。  
理由は、Dockerコンテナに接続してファイルを作成するときはrootユーザーのためファイルの権限が `-rw-r--r-- 1 root root` になります。  
しかしVSCodeではWSLのデフォルトユーザー(rootユーザーではない)でファイルを編集するため書き込み制限に引っかかるためです。  
これを解決するにはDockerコンテナにWSLデフォルトユーザーとして接続するか、WSLデフォルトユーザーをrootユーザーと同じグループにすればいいのかなと思いました。  

DockerコンテナにWSLデフォルトユーザーとして接続してみます。(`1000` はWSLデフォルトユーザーのuid)

```bash
$ docker compose exec --user 1000 web bash
```

ファイルを作って権限を確認します。

```bash
$ php artisan make:command SendEmails

 INFO  Console command [app/Console/Commands/SendEmails.php] created successfully.

$ ls -l app/Console/Commands/SendEmails.php
-rw-r--r-- 1 1000 root ～略～
```

ファイルの権限が `-rw-r--r-- 1 1000 root` になりました。  
Dockerコンテナから出てWSL上で確認します。(WSLデフォルトユーザー名が `username` のとき)

```bash
$ ls -l app/Console/Commands/SendEmails.php
-rw-r--r-- 1 username root ～略～
```

WSLデフォルトユーザーで書き込み可能になっていることが確認できました。  
これでVSCodeから編集できます。(もっと他にいい方法がありそう)
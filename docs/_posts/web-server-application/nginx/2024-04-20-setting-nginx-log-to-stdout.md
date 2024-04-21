---
layout: my-post
title: "NGINXのログを標準出力に設定する"
date: 2024-04-20 00:00:00 +0000
categories: web-server-application nginx
title_eng: setting-nginx-log-to-stdout
---

NGINXのアクセスログを標準出力に、エラーログを標準エラー出力に出力するように設定します。  

今回はDockerコンテナ上のNGINXを設定します。NGINXの公式Dockerイメージではなく、Amazon Linux 2023のイメージにNGINXをインストールしたものを使用しています。

## 参考ページ
- [コンテナ内のプロセスのログ出力先を標準出力/標準エラー出力に設定する方法](https://qiita.com/sshota0809/items/a86cd3379f88fb5cd1b8)

## 前提
- NGINXをインストールしたDockerコンテナがある。  
WSL上のDockerコンテナでNGINXを動かす方法については[こちら](/web-application-framework/laravel/running-laravel-project-on-nginx)をご覧ください。

## 環境
- Windows 10 64ビット
- WSL2 + Ubuntu 22.04.3 LTS
- Docker Engine 26.0.0
- Amazon Linux 2023(DockerコンテナのOS)
- NGINX 1.24.0

## 設定の流れ
1. [ログファイルの場所を確認する。](#1-ログファイルの場所を確認する)
2. [標準出力のシンボリックリンクを作成する。](#2-標準出力のシンボリックリンクを作成する)
3. [設定を確認する。](#3-設定を確認する)

## 1. ログファイルの場所を確認する
NGINXのアクセスログとエラーログのファイルの場所を確認します。  

NGINXの設定ファイルである `nginx.conf` を開き、`access_log` と `error_log` を確認します。  
Amazon Linux 2023の場合、`nginx.conf` はデフォルトで `/etc/nginx` ディレクトリにありました。

`access_log` と `error_log` は以下の通りでした。
```
access_log  /var/log/nginx/access.log  main;
error_log /var/log/nginx/error.log notice;
```
`access_log` の出力場所は `/var/log/nginx/access.log` で、`error_log` の出力場所は `/var/log/nginx/error.log` です。  
`access_log` の後ろにある `main` は使用するログフォーマットです。`nginx.conf` の `log_format` にデフォルトで `main` が定義されており、それを指定しています。  
`error_log` の後ろにある `notice` は出力する最低レベルです。`notice` 以上のレベルのログが出力されます。

ログファイルの場所を確認できました。

## 2. 標準出力のシンボリックリンクを作成する
標準出力にログを出力するため、ログファイルを標準出力のシンボリックリンクにします。  

アクセスログファイルを標準出力のシンボリックリンクにし、エラーログを標準エラー出力のシンボリックリンクにします。  
NGINXをインストールするDockerイメージのDockerfileを以下のように変更します。

`Dockerfile` :
```dockerfile
FROM amazonlinux:2023

～NGINXをインストールする処理～

# シンボリックリンクを作成する。リンク名をログファイル名にする。
RUN ln -s /dev/stdout /var/log/nginx/access.log
RUN ln -s /dev/stderr /var/log/nginx/error.log
```

ログファイルを標準出力のシンボリックリンクにできました。

## 3. 設定を確認する
標準出力にログが出力されるか確認します。  

NGINXをインストールするDockerコンテナを起動し、ブラウザなどからNGINXサーバーにアクセスします。  
その後Dockerのログを確認します。以下を実行します。  
```bash
$ docker logs <コンテナID>
```
以下のようにNGINXのアクセスログが出力されたら成功です。
```
192.168.224.1 - - [20/Apr/2024:05:50:47 +0000] "GET / HTTP/1.1" 200 34063 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/123.0.0.0 Safari/537.36" "-"
192.168.224.1 - - [20/Apr/2024:05:50:47 +0000] "GET / HTTP/1.1" 200 34063 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/123.0.0.0 Safari/537.36" "-"
```
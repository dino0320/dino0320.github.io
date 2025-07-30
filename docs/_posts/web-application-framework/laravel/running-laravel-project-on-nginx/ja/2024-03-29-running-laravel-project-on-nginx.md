---
layout: my-post
title: "LaravelプロジェクトをNGINXで動かす"
date: 2024-03-29 00:00:00 +0000
categories: web-application-framework laravel
page_name: running-laravel-project-on-nginx
lang: ja
---

Dockerを使ってLaravelプロジェクトを動かす環境を構築します。  
NGINX + php-fpmでLaravelプロジェクトを動かします。

## NGINXとは
NGINXはオープンソースなWebサーバーソフトウェアです。  
NGINXをインストールしたコンピューターはWebサーバーの動きができるようになります。

## 環境
- Ubuntu 22.04.3 LTS (WSLで起動している)
- Docker Engine 26.0.0
- Composer 2.7.2
- Laravel 11

## 環境構築の流れ
1. [Laravelプロジェクトを作成する。](#1-laravelプロジェクトを作成する)
2. [設定ファイルを作成する。](#2-設定ファイルを作成する)
3. [Dockerの設定ファイルを作成する。](#3-dockerの設定ファイルを作成する)
4. [Laravelプロジェクトを確認する。](#4-laravelプロジェクトを確認する)

## 1. Laravelプロジェクトを作成する
Laravelプロジェクトの作成については[こちら](/web-application-framework/laravel/creating-laravel-project-on-linux)をご覧ください。

## 2. 設定ファイルを作成する
Laravelプロジェクトのディレクトリ以下で下記の設定ファイルを作成します。  
記載方法について、Laravelプロジェクトのルートディレクトリをルートディレクトリ(`/`)とします。

### nginx.repo
Amazon Linux 2023用にyumリポジトリをセットアップするための設定ファイルです。[こちら](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#installing-prebuilt-amazon-linux-packages)を参考にしています。  
`/docker/web/nginx` ディレクトリに作成します。

`/docker/web/nginx/nginx.repo` :
```
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/amzn/2023/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/amzn/2023/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
```

### default.conf
LaravelプロジェクトのためのNGINXの設定ファイルです。[こちら](https://laravel.com/docs/11.x/deployment#nginx)を参考にしています。  
`server_name` を `localhost` に変更しています。動作確認が目的なので、少し変更を加えるだけにしています。  
`/docker/web/nginx/conf.d` ディレクトリに作成します。

`/docker/web/nginx/conf.d/default.conf` :
```conf
server {
    listen 80;
    listen [::]:80;
    server_name localhost;
    root /srv/example.com/public;
 
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";
 
    index index.php;
 
    charset utf-8;
 
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
 
    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }
 
    error_page 404 /index.php;
 
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }
 
    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

### zzz-www.conf
php-fpmの追加の設定ファイルです。`.sock` ファイルのパスを上書きするために作成します。  
`/docker/web/php/php-fpm.d` ディレクトリに作成します。

`/docker/web/php/php-fpm.d/zzz-www.conf` :
```conf
[www]
listen = /var/run/php/php8.2-fpm.sock
```

## 3. Dockerの設定ファイルを作成する
Laravelプロジェクトのディレクトリ以下に下記の設定ファイルを作成します。  
記載方法について、Laravelプロジェクトのルートディレクトリをルートディレクトリ(`/`)とします。

### docker-compose.yml
Docker Composeの設定ファイルです。  
`/` ディレクトリに作成します。

`/docker-compose.yml` :
```yml
services:
  web:
    build: ./docker/web
    volumes:
      - .:/srv/example.com
    ports:
      - "8080:80"
    command: bash -c "chmod 755 /srv/example.com/docker/web/start.sh && /srv/example.com/docker/web/start.sh"
```

### Dockerfile
webのDockerfileです。(NGINX + php-fpmをインストールするDockerコンテナ)  
Dockerイメージは、今回は後でAWSにデプロイすることを考えて `amazonlinux` にしています。  
`/docker/web` ディレクトリに作成します。

`/docker/web/Dockerfile` :
```dockerfile
FROM amazonlinux:2023

# NGINXインストール
RUN yum -y install yum-utils
COPY nginx/nginx.repo /etc/yum.repos.d/nginx.repo
RUN yum -y install nginx-1.24.0
COPY nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf

# php-fpmとphp-mysqlndインストール
RUN yum -y install php8.2-fpm php8.2-mysqlnd
# ディレクトリを作っておかないとエラーになる
RUN mkdir /run/php-fpm
RUN mkdir /var/run/php
COPY php/php-fpm.d/zzz-www.conf /etc/php-fpm.d/zzz-www.conf
```

### start.sh
Dockerコンテナ起動時に実行される `start.sh` を作成します。

`start.sh`:

```sh
#!/bin/bash

set -euxo pipefail

PROJECT_PATH=/srv/example.com

# Nginxユーザーに以下のファイルにアクセスする権限を与える
chmod 777 "$PROJECT_PATH/storage/logs"
chmod 777 "$PROJECT_PATH/storage/framework/views"
chmod 777 "$PROJECT_PATH/database/database.sqlite"

# php-fpmとNGINX起動
# nginxは「-g "daemon off;"」でフォアグラウンド実行になり、コンテナが自動的に終了しなくなる
php-fpm
nginx -g "daemon off;"
```

## 4. Laravelプロジェクトを確認する
Dockerコンテナを立ち上げてLaravelプロジェクトを確認します。  
以下を実行します。
```
$ cd <Laravelプロジェクトのルートディレクトリのパス>
$ docker compose up --build -d
```
コンテナはWSL上のUbuntuで立ち上げているので、Windows側から `http://localhost:8080` にアクセスします。

![Laravelプロジェクトの画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Laravelプロジェクトの画面")

Laravelプロジェクトが確認できました。
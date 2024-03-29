---
layout: my-post
title: "LaravelプロジェクトをNGINXで動かす"
date: 2024-03-28 00:00:00 +0000
categories: programming php
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
2. [Dockerの設定ファイルを作成する。](#2-php-fpmの設定ファイルを取得する)
3. [設定ファイルを作成・変更する。](#3-設定ファイルを作成変更する)
4. [Dockerの設定ファイルを作成する。](#4-dockerの設定ファイルを作成する)
5. [Laravelプロジェクトを確認する。](#5-laravelプロジェクトを確認する)

## 1. Laravelプロジェクトを作成する
Laravelプロジェクトの作成については[こちら](/programming/php/creating-laravel-project-on-linux)をご覧ください。

## 2. 必要な設定ファイルを取得する
インストールするときなどに生成される設定ファイルを変更して使用したいので、NGINXとphp-fpmをインストールして必要な設定ファイルを取得します。  
以下を実行して設定ファイルを取得するためのコンテナを立ち上げ、接続します。
```
$ cd <Laravelプロジェクトのルートディレクトリのパス>
$ docker run -it --rm --name amazonlinux_for_get_config -w /app -v `pwd`:/app amazonlinux:2023 bash
```
Laravelプロジェクトのディレクトリ以下に設定ファイルを保存するため、コンテナ内の `/app` ディレクトリに `<Laravelプロジェクトのルートディレクトリのパス>` をマウントしています。  
Dockerイメージは、今回は後でAWSにデプロイすることを考えてamazonlinuxにしています。設定ファイルを取得するだけなので、ここではamazonlinux以外のイメージを使用しても問題ないかもしれません。

コンテナ内で以下を実行します。  
Amazon Linux 2023用にyumリポジトリをセットアップするための設定ファイル `nginx.repo` を作成します。
```
# yum -y install vim    # vimをインストール
# vim /etc/yum.repos.d/nginx.repo    # 設定ファイルを作成
```
設定ファイル `nginx.repo` の内容は以下になります。[こちら](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#installing-prebuilt-amazon-linux-packages)を参考にしています。

`/etc/yum.repos.d/nginx.repo` :
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
NGINXをインストールします。
```
# yum -y install yum-utils
# yum -y install nginx-1.24.0
```
php-fpmをインストールします。
```
# yum -y install php8.2-fpm
```
必要な設定ファイルをコピーします。
```
# mkdir -p ./docker/web/nginx/conf.d    # 設定ファイルを保存するディレクトリを作成
# cp /etc/yum.repos.d/nginx.repo ./docker/web/nginx/nginx.repo    # Amazon Linux 2023用にyumリポジトリをセットアップするための設定ファイルをコピー
# cp /etc/nginx/conf.d/php-fpm.conf ./docker/web/nginx/conf.d/php-fpm.conf    # NGINXのphp-fpmの設定ファイルをコピー
# mkdir ./docker/web/php    # 設定ファイルを保存するディレクトリを作成
# cp /etc/php-fpm.d/www.conf ./docker/web/php/www.conf    # php-fpmの設定ファイルをコピー
```
コンテナから抜け、設定ファイルを確認します。
```
# exit
$ ls ./docker/web/nginx/
conf.d  nginx.repo
$ ls ./docker/web/nginx/conf.d/
php-fpm.conf
$ ls ./docker/web/php
www.conf
```
設定ファイルを取得できました。

## 3. 設定ファイルを作成・変更する
Laravelプロジェクトのディレクトリ以下で下記の設定ファイルを作成・変更します。  
記載方法について、Laravelプロジェクトのルートディレクトリをルートディレクトリ(`/`)とします。
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

### php-fpm.conf
NGINXのphp-fpmの設定ファイルです。  
取得した `/docker/web/nginx/conf.d/php-fpm.conf` の以下の箇所を変更します。

`/docker/web/nginx/conf.d/php-fpm.conf` :
```conf
# PHP-FPM FastCGI server
# network or unix domain socket configuration

upstream php-fpm {
        server unix:/var/run/php/php8.2-fpm.sock;    # パスを変更
}
```

### www.conf
php-fpmの設定ファイルです。  
取得した `/docker/web/php/www.conf` の以下の3箇所を変更します。

`/docker/web/php/www.conf` :
```conf
;listen = /run/php-fpm/www.sock    # コメントアウト
listen = /var/run/php/php8.2-fpm.sock    # 追加
```
```conf
;listen.owner = nobody    # コメントアウト
;listen.group = nobody    # コメントアウト
listen.owner = nginx    # 追加
listen.group = nginx    # 追加
```
```conf
;listen.acl_users = apache,nginx    # コメントアウト
```

## 4. Dockerの設定ファイルを作成する
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
      - ./docker/web/nginx/conf.d:/etc/nginx/conf.d
    ports:
      - "8080:80"
```

### Dockerfile
webのDockerfileです。(NGINX + php-fpmをインストールするDockerコンテナ)  
`/docker/web` ディレクトリに作成します。

`/docker/web/Dockerfile` :
```dockerfile
FROM amazonlinux:2023

# NGINXインストール
RUN yum -y install yum-utils
COPY nginx/nginx.repo /etc/yum.repos.d/nginx.repo    # 設定ファイルをコピー
RUN yum -y install nginx-1.24.0

# php-fpmとphp-mysqlndインストール
RUN yum -y install php8.2-fpm php8.2-mysqlnd
RUN mkdir /run/php-fpm    # ディレクトリを作っておかないとエラーになる
RUN mkdir /var/run/php    # ディレクトリを作っておかないとエラーになる
COPY php/www.conf /etc/php-fpm.d/www.conf    # 設定ファイルをコピー

# php-fpmとNGINX起動
# nginxは「-g "daemon off;"」でフォアグラウンド実行になり、コンテナが自動的に終了しなくなる
CMD bash -c '/usr/sbin/php-fpm && /usr/sbin/nginx -g "daemon off;"'
```

## 5. Laravelプロジェクトを確認する
Dockerコンテナを立ち上げてLaravelプロジェクトを確認します。  
以下を実行します。
```
$ cd <Laravelプロジェクトのルートディレクトリのパス>
$ docker compose up --build -d
```
コンテナはWSL上のUbuntuで立ち上げているので、Windows側から `http://localhost:8080` にアクセスします。

![Laravelプロジェクトの画面](/assets/images/application/server/running-laravel-project-on-nginx/image1.png "Laravelプロジェクトの画面")

Laravelプロジェクトが確認できました。
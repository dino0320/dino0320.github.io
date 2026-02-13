---
layout: my-post
title: "Docker ComposeでNestJS + Nginxのリバースプロキシ構成を作る方法"
date: 2026-02-13 00:00:00 +0000
categories: web-application-framework nestjs
page_name: how_to_set_up_nestjs_and_nginx_reverse_proxy_with_docker_compose
lang: ja
image: /assets/images/web-application-framework/nestjs/how_to_set_up_nestjs_and_nginx_reverse_proxy_with_docker_compose/image1.png
---

Docker Composeを使用してリバースプロキシ構成のNestJS + Nginx環境を構築する方法をまとめました。

*この構成は開発を目的としています。*  
*本番環境では、NginxとNestJSアプリケーションを別々のコンテナに分離し、適切なプロセス管理を行うことを推奨します。*

*本記事では、AWS環境との整合性を保ち、OSレベルで柔軟なカスタマイズを可能にするため、Amazon Linuxをベースイメージとして使用しています。*  
*一般的には公式のNode.jsやNginxイメージが利用されますが、本構成ではよりインフラ寄りのアプローチを採用しています。*

![サムネイル](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "サムネイル")

## 参照
- [NestJS: First steps](https://docs.nestjs.com/first-steps)
- [リバースプロキシとは？](https://www.cloudflare.com/ja-jp/learning/cdn/glossary/reverse-proxy/)
- [Amazon Linux 2023 packages](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#amazon-linux-2023-packages)

## 環境
- Ubuntu 24.04.3 LTS (WSL2ディストリビューション)
- Docker Engine 28.4.0
- Amazon Linux 2023(DockerコンテナのOS)
- Node.js 24.13.1
- NestJS 11.0.1
- NGINX 1.28.2

## セットアップ手順
1. [NestJSプロジェクトを作成する](#1-nestjsプロジェクトを作成する)
2. [設定ファイルを作成する](#2-設定ファイルを作成する)
3. [Docker設定ファイルを作成する](#3-docker設定ファイルを作成する)
4. [NestJSプロジェクトを確認する](#4-nestjsプロジェクトを確認する)

## 1. NestJSプロジェクトを作成する
まず、NestJSプロジェクトを作成するためにDockerコンテナを起動します。  
以下のコマンドを実行してください。`<path-to-nestjs-project-dir>` はNestJSプロジェクトを作成したいディレクトリを指します。

```bash
$ cd <path-to-nestjs-project-dir>
$ docker run -it --rm --name node_to_install_nestjs -w /app -v `pwd`:/app node:24.13.1 bash
```

このコマンドは、`node:24.13.1` イメージから `node_to_install_nestjs` という名前のコンテナを起動し、`bash` でそのコンテナに接続します。

NestJS CLIをインストールしてプロジェクトを作成します。  
コンテナ内で以下を実行してください。

```bash
$ npm i -g @nestjs/cli
$ nest new <project-name>
```

`<project-name>` は任意のプロジェクト名に置き換えてください。

成功すると、現在のディレクトリにNestJSプロジェクトが作成されます。  
`ls` コマンドで確認します。

```bash
$ ls
<project-name>
```

`docker run` コマンドでは `-v` オプションでホスト側のディレクトリをコンテナにマウントしているため、ホスト側からもプロジェクトを確認できます。

```bash
$ exit # コンテナを終了する
$ ls
<project-name>
```

## 2. 設定ファイルを作成する
NestJSプロジェクトディレクトリ内に以下の設定ファイルを作成します。  
カレントディレクトリはプロジェクトのルートディレクトリです。

### nginx.repo
この設定ファイルはAmazon Linux 2023用のYUMリポジトリを設定するものです。  
[Amazon Linux 2023 packages](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#amazon-linux-2023-packages)を元にしています。

`docker/web/nginx` ディレクトリに作成します。

`docker/web/nginx/nginx.repo` :
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
NestJSプロジェクト用のNginx設定ファイルを `docker/web/nginx/conf.d` ディレクトリに作成します。

`docker/web/nginx/conf.d/default.conf` :
```conf
server {
    listen 80;
    listen [::]:80;
    server_name localhost;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

NestJSは `src/main.ts` ファイルで定義されたポート（デフォルトは `3000`）でサーバーを起動します。

この構成ではNginxがリバースプロキシとして動作します。  
ユーザーがアプリケーションにリクエストを送ると、Nginxがそれを受け取り、NestJS アプリケーションへ転送します。

![アクセスの流れ](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image2.png "アクセスの流れ")

## 3. Docker設定ファイルを作成する
以下のDocker関連設定ファイルを作成します。  
カレントディレクトリはプロジェクトのルートディレクトリです。

### docker-compose.yml
Docker Composeの設定ファイルです。  
プロジェクトのルートディレクトリに作成してください。

`docker-compose.yml` :
```yml
services:
  web:
    build: ./docker/web
    volumes:
      - .:/srv/example.com
    ports:
      - "8080:80"
    command: bash -c "chmod 755 /srv/example.com/docker/web/start.sh &&
             /srv/example.com/docker/web/start.sh"
```

### Dockerfile
この `Dockerfile` はDockerサービスのweb（Nginx + Node.js）を定義します。  
将来的なAWSへのデプロイを想定し、`amazonlinux` イメージを使用しています。

`docker/web` ディレクトリに作成してください。

`docker/web/Dockerfile` :
```dockerfile
FROM amazonlinux:2023

# Nginxをインストールする
RUN yum -y install yum-utils
COPY nginx/nginx.repo /etc/yum.repos.d/nginx.repo
RUN yum -y install nginx-1.28.2
COPY nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf

# Node.jsをインストールする
RUN yum -y install tar
RUN touch ~/.bashrc
COPY install-npm.sh /tmp/install-npm.sh
RUN chmod 755 /tmp/install-npm.sh
RUN /tmp/install-npm.sh 24.13.1
```

### install-npm.sh
Node.jsとnpmをインストールするための `install-npm.sh` を作成します。  
`docker/web` ディレクトリに配置してください。

`docker/web/install-npm.sh`:

```sh
#!/bin/bash

set -euxo pipefail

curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash

source ~/.bashrc

VERSION=$1
nvm install $VERSION
```

### start.sh
Dockerコンテナ起動時に実行される `start.sh` を作成します。  
`docker/web` ディレクトリに配置してください。

`docker/web/start.sh`:

```sh
#!/bin/bash

set -euxo pipefail

# nginxユーザーをrootグループに追加（アクセス権限のため）
usermod -aG root nginx

# npmパッケージをインストールする
npm ci

# Nginxを起動する
# -g "daemon off;" を指定することで、Nginxをフォアグラウンドで実行し、コンテナが自動終了するのを防ぐ
nginx -g "daemon off;"
```

## 4. NestJSプロジェクトを確認する
Dockerコンテナを起動します。

```bash
$ cd <path-to-your-nestjs-project-root>
$ docker compose up --build -d
```

コンテナ内でNestJSサーバーを起動します。

```bash
$ docker compose exec web bash # コンテナに接続する
$ cd /srv/example.com
$ npm run start
```

ブラウザで `http://localhost:8080` にアクセスします。

![NestJSアプリの画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image3.png "NestJSアプリの画面")

正しく動作していれば `Hello World!` メッセージが表示されます。
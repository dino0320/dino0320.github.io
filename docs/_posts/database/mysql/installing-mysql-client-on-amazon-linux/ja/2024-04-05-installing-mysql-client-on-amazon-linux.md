---
layout: my-post
title: "Amazon Linux 2023にMySQLクライアントをインストールする"
date: 2024-04-05 00:00:00 +0000
categories: database mysql
page_name: installing-mysql-client-on-amazon-linux
lang: ja
image: /assets/images/database/mysql/installing-mysql-client-on-amazon-linux/image1.png
---

Amazon Linux 2023のDockerイメージから作成したDockerコンテナにMySQLクライアントをインストールします。  
[こちらのページ](https://dev.classmethod.jp/articles/install-mysql-client-to-amazon-linux-2023/)を参考にさせていただいています。  

![サムネイル](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "サムネイル")

## MySQLクライアントとは
MySQLサーバーへの接続・操作ができるようになるツールです。

## 環境
- Ubuntu 22.04.3 LTS (WSLで起動している)
- Docker Engine 26.0.0

## インストールの流れ
1. [MySQLクライアントをインストールする。](#1-mysqlクライアントをインストールする)
2. [インストールを確認する。](#2-インストールを確認する)

## 1. MySQLクライアントをインストールする
Amazon Linux 2023の `Dockerfile` を作成し、MySQLクライアントインストールの行を追加します。  

`Dockerfile` :
```dockerfile
FROM amazonlinux:2023

# MySQLクライアントインストール
RUN rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
RUN yum -y install https://repo.mysql.com/mysql80-community-release-el9-1.noarch.rpm
RUN yum -y install mysql-community-client-8.0.34
```
EL9ベースシステム用のyumリポジトリを追加していますが、[参考サイト様](https://dev.classmethod.jp/articles/install-mysql-client-to-amazon-linux-2023/)によるとFedora向けのインストールはできないみたいです。(AL2023にはFedora 34、35、36のコンポーネントが含まれている)

GPGキーは期限があるそうなので、この記事をご覧になった時期によってはこのキーではだめかもしれません。  
GPGキーとyumリポジトリは[こちら](https://repo.mysql.com)から確認できました。

今回は今後AWSにデプロイすることを考えて、Amazon Auroraに合わせてMySQLサーバーのバージョンを `8.0.34` にします。そのためMySQLクライアントのバージョンも `8.0.34` をインストールしています。(サーバーのバージョンとクライアントのバージョンが同じなら問題ないのかどうかはわかりませんが…)  

## 2. インストールを確認する
インストールしたMySQLクライアントの動作確認を行います。  
作成したAmazon Linux 2023の `Dockerfile` と同じディレクトリに以下の `docker-compose.yml` を作成します。

`docker-compose.yml` :
```yml
services:
  mysql_client:
    build: .
    tty: true    # Dockerコンテナが自動で終了しないように追加
  mysql_server:
    image: mysql:8.0.34
    environment:
      MYSQL_ROOT_PASSWORD: root
```
同じディレクトリで以下を実行します。  
Dockerコンテナを立ち上げてmysql_client(Amazon Linux 2023の方)に接続しています。
```bash
$ docker compose up -d
$ docker compose exec mysql_client bash
```
コンテナ内で以下を実行します。
```
# mysql -u root -p -h mysql_server
```
rootユーザーのパスワードを聞かれるので `root` を入力します。  
`docker-compose.yml` で設定した `MYSQL_ROOT_PASSWORD` の値です。

プロンプトが `mysql>` に変わったら接続成功です。  
MySQLクライアントのインストールを確認できました。
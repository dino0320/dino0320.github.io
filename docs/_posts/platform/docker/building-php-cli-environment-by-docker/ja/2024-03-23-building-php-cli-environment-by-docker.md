---
layout: my-post
title: "DockerでPHPをCLIで実行できる環境を構築する"
date: 2024-03-23 00:00:00 +0000
categories: platform docker
lang: ja
---

DockerでPHPをCLIで実行できる環境を構築します。

## 環境
- Ubuntu 22.04.3 LTS (WSLで起動している)
- Docker Engine 26.0.0

## 前提
- Dockerをインストールしている。  
WindowsでのDoker Engineのインストールについては[こちら](/platform/docker/installing-docker-engine-on-windows)をご覧ください。

## 環境構築の流れ
1. [PHPファイルを作成する。](#1-phpファイルを作成する)
2. [PHPコンテナを立ち上げる。](#2-phpコンテナを立ち上げる)
3. [PHPファイルを実行する。](#3-phpファイルを実行する)

## 1. PHPファイルを作成する
実行するPHPのファイルを作成します。  
以下を実行して"Hello World"を出力する `sample.php` を作成します。
```
$ echo '<?php echo "Hello World" . PHP_EOL;' > sample.php
```
ファイルの中身は以下の通りです。
```php
<?php echo "Hello World" . PHP_EOL;
```

## 2. PHPコンテナを立ち上げる
作成したPHPファイルがあるディレクトリに移動します。  
以下を実行します。
```
$ docker run -it --rm --name php-cli -w /app -v `pwd`:/app php:8.2-cli bash
```

`php:8.2-cli` のイメージから `php-cli` という名前のDockerコンテナを立ち上げ、`bash` コマンドでコンテナに接続しています。コンテナ名は任意です。([`docker run` コマンドについて](/platform/docker/about-docker-commands#docker-run))

## 3. PHPファイルを実行する
PHPコンテナに接続した状態で以下を実行します。  
PHPファイルがあるディレクトリをPHPコンテナ内の `/app` ディレクトリにマウントしてるので、`/app` 以下に `sample.php` があるはずです。
```
/app# php sample.php
Hello World
```
作成したPHPファイルを実行できました。  
`exit` コマンドでコンテナとの接続を切ると、`docker run` コマンドの `rm` オプションの効果でコンテナが削除されます。  
```
/app# exit
exit
$ docker ps -a # コンテナは削除されたため表示されない
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```
再びコンテナを作成し、PHPファイルを実行したい場合は[手順2](#2-phpコンテナを立ち上げる)からやり直してください。

#### PHPコンテナを立ち上げるタイミングでPHPファイルを実行する
PHPコンテナを立ち上げるタイミングでPHPファイルを実行する場合は、[手順2](#2-phpコンテナを立ち上げる)で実行した `docker run` コマンドの `bash` を `php sample.php` に変更して実行します。
```
$ docker run -it --rm --name php-cli -w /app -v `pwd`:/app php:8.2-cli php sample.php
Hello World
```
コンテナ起動時に `bash` コマンドを実行する代わりに `php sample.php` を実行します。

#### PHPファイルを作成せずにPHPを実行する
PHPファイルを作成せずにPHPを実行する場合は、[手順1](#1-phpファイルを作成する)を行わず、[手順2](#2-phpコンテナを立ち上げる)で実行した `docker run` コマンドの `bash` を `php -r "echo 'Hello World' . PHP_EOL;"` に変更して実行します。
```
$ docker run -it --rm --name php-cli -w /app -v `pwd`:/app php:8.2-cli php -r "echo 'Hello World' . PHP_EOL;"
Hello World
```
コンテナ起動時に `bash` コマンドを実行する代わりに `php -r "echo 'Hello World' . PHP_EOL;"` を実行します。  
`php` コマンドの `r` オプションで、コマンドラインのみでPHPを実行できます。
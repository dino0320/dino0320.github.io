---
layout: my-post
title: "DockerでPHPをCLIで実行できる環境を構築する"
date: 2024-03-23 00:00:00 +0000
categories: platform docker
---

DockerでPHPをCLIで実行できる環境を構築します。

## 環境
- Ubuntu 22.04.3 LTS (WSLで起動している)

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
以下を実行してPHPのコンテナを立ち上げ、コンテナ内のbashを起動してコンテナに接続します。
```
$ docker run -it --rm --name php-cli -w /app -v `pwd`:/app php:8.2-cli bash
```

`docker run` コマンドはDockerイメージからコンテナを立ち上げるコマンドです。  
使い方は以下の通りです。
```
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

今回実行したコマンドについて説明します。  
以下は引数の説明です。

|引数|説明|
|----|----|
|IMAGE|Dockerイメージを指定します。今回は `php:8.2-cli` を指定しています。このイメージはPHP 8.2をコマンドラインから実行するために必要なツールを含んでいます。|
|COMMAND|コンテナ起動後に実行するコマンドを指定できます。今回は `bash` コマンドを指定し、コンテナに接続するようにしています。|

以下はオプションの説明です。

|オプション|説明|
|----|----|
|i|標準入力を開き続けます。コンテナへの入力ができるようになります。|
|t|疑似TTYを割り当てます。TTYはユーザーの入出力デバイスで、コンテナに接続したときに入力を受け付けたり出力を表示したりできるようになります。また、TTYのプロセスが動くことで、コンテナ接続中はコンテナが終了することを防ぎます。|
|rm|コンテナが終了するときにコンテナを自動的に削除します。|
|name|コンテナに名前を付けます。|
|w|コンテナ内のワーキングディレクトリ(接続したときにいるディレクトリ)です。|
|v|コンテナにホストのディレクトリをマウントします。今回は `pwd` コマンドでホストの現在のディレクトリを出力し、コンテナ内の `/app` ディレクトリにマウントしています。|

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
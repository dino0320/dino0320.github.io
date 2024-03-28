---
layout: my-post
title: "Laravelをインストールする"
date: 2024-03-28 00:00:00 +0000
categories: programming php
---

Linux環境でLaravelをインストールします。  
Dockerを使ってインストールします。

## Laravelとは
PHP用のWebアプリケーションフレームワークの1つです。  
WebアプリケーションフレームワークはWebアプリケーションの開発が楽になる枠組みのことです。  
Laravelはサーバー側のフレームワークになります。

## 環境
- Ubuntu 22.04.3 LTS (WSLで起動している)
- Docker Engine 26.0.0
- Composer 2.7.2

## インストールの流れ
1. [PHPコンテナを立ち上げる。](#1-phpコンテナを立ち上げる)
2. [Composerをインストールする。](#2-composerをインストールする)
3. [Laravelをインストールする。](#3-laravelをインストールする)
4. [Laravelプロジェクトを確認する。](#4-laravelプロジェクトを確認する)

## 1. PHPコンテナを立ち上げる
LaravelをインストールするためのDockerコンテナを立ち上げます。  
以下を実行します。  
`<Laravelプロジェクトを作成するディレクトリのパス>` はLaravelを使ったプロジェクトを置くディレクトリのパスです。
```
$ cd <Laravelプロジェクトを作成するディレクトリのパス>
$ docker run -it --rm --name php_to_install_laravel -w /app -v `pwd`:/app php:8.3 bash
```
``docker run -it --rm --name php_to_install_laravel -w /app -v `pwd`:/app php:8.3 bash`` で `php:8.3` のイメージから `php_to_install_laravel` という名前のDockerコンテナを立ち上げ、`bash` コマンドでコンテナに接続しています。コンテナ名は任意です。

## 2. Composerをインストールする
Laravelのインストールに必要なので、接続したコンテナ内でComposerをインストールします。  
Composerのインストールについては[こちら](/programming/php/installing-composer-on-linux)をご覧ください。

## 3. Laravelをインストールする
Laravel(バージョン11)をインストールしてプロジェクトを作成します。   
接続したコンテナ内で以下を実行します。  
`<Laravelプロジェクト名>` はLaravelを使ったプロジェクトの名前です。任意の名前を入力します。
```
# composer create-project laravel/laravel:^11.0 <Laravelプロジェクト名>
```
`unzip` か `7z` がないとLaravelをインストールするときにエラーになると思います。  
`unzip` も `7z` もない場合は以下を実行し、再度Laravelをインストールします。
```
# apt update
# apt install unzip
# composer create-project laravel/laravel:^11.0 <Laravelプロジェクト名> # Laravelの再インストール
```
成功すると現在のディレクトリにLaravelのプロジェクトが作成されているはずです。  
以下を実行して確認します。  
```
# ls
<Laravelプロジェクト名>
```
`docker run` コマンドの `v` オプションでホストのディレクトリをコンテナのディレクトリにマウントしているので、ホスト側でもLaravelのプロジェクトを確認できます。  
以下を実行します。
```
# exit # コンテナの接続を切る
$ ls
<Laravelプロジェクト名>
```

## 4. Laravelプロジェクトを確認する
作成したLaravelプロジェクトをローカルで確認します。  
以下を実行します。   
`<Laravelプロジェクトの親ディレクトリのパス>` はLaravelを使ったプロジェクトの親ディレクトリのパスです。
```
$ cd <Laravelプロジェクトの親ディレクトリのパス>
$ docker run -it --rm --name php_to_confirm_laravel -w /app -v `pwd`:/app -p 8000:8000 php:8.3 bash
```
``docker run -it --rm --name php_to_confirm_laravel -w /app -v `pwd`:/app -p 8000:8000 php:8.3 bash`` で `php:8.3` のイメージから `php_to_confirm_laravel` という名前のDockerコンテナを立ち上げ、`bash` コマンドでコンテナに接続しています。  
`docker run` コマンドの `p` オプションでホストとコンテナのポートを接続しています。

コンテナ内で以下を実行します。
```
# cd <Laravelプロジェクト名>
# php artisan serve --host 0.0.0.0
```
`php artisan serve --host 0.0.0.0` でローカルサーバーが起動します。  
`host` オプションはサーバーのIPアドレスを指定しますが、`0.0.0.0` にすることでコンテナの外からアクセスできるようにしています。

コンテナはWSL上のUbuntuで立ち上げているので、Windows側から `http://localhost:8000` にアクセスします。

![Laravelプロジェクトの画面](/assets/images/programming/php/installing-laravel-on-linux/image1.png "Laravelプロジェクトの画面")

Laravelプロジェクトが確認できました。
---
layout: my-post
title: "WSL + Docker上のPHPコードをVSCodeでXdebugする"
date: 2024-04-02 00:00:00 +0000
categories: ide vscode
page_name: xdebug-php-with-vscode-on-wsl-and-docker
lang: ja
---

WSL上のDockerコンテナで動くPHPのコードをVisual Studio Code(VSCode)でXdebugできるようにします。

## 環境
- Windows 10 64ビット
- Ubuntu 22.04.3 LTS (WSLで起動している)
- Docker Engine 26.0.0
- PHP 8.2.15
- Visual Studio Code 1.87.2

## 前提
- WSL上のDockerコンテナで動くPHPのコードがある。(Docker Compose使用)  
WSL上のDockerコンテナで動くLaravelプロジェクトの構築については[こちら](/web-application-framework/laravel/running-laravel-project-on-nginx)をご覧ください。
- Visual Studio Codeをインストールしている。

## 設定の流れ
1. [VSCodeでWSLに接続する。](#1-vscodeでwslに接続する)
2. [Xdebugをインストールする。](#2-xdebugをインストールする)
3. [VSCodeの拡張機能であるPHP Debugをインストールする。](#3-vscodeの拡張機能であるphp-debugをインストールする)
4. [Xdebugを確認する。](#4-xdebugを確認する)

## 1. VSCodeでWSLに接続する
VSCodeでWSLに接続する方法については[こちら](/ide/vscode/connecting-to-wsl-with-vscode)をご覧ください。

## 2. Xdebugをインストールする
DockerコンテナにXdebugをインストールします。  

`docker-compose.yml` にDockerを実行しているホストの定義を追加します。  
Docker Desktopを使用している場合はこの設定は必要ないみたいです。  

`docker-compose.yml` :
```yml
services:
  web:
    build: ./docker/web
    volumes:
      - .:/srv/example.com
    ports:
      - 8080:80
    extra_hosts:
      - host.docker.internal:host-gateway    # ホストの定義を追加する
```
PHPの設定にXdebugの設定を追加します。  
以下のファイルを作成します。  

`99-xdebug.ini` : 
```ini
[xdebug]
zend_extension=/usr/lib64/php8.2/modules/xdebug.so    # .soファイルがあるパスを指定する
xdebug.mode = debug    # リモートでバッグが有効になる
xdebug.start_with_request = yes    # リモートでバッグが有効になる
xdebug.client_host = host.docker.internal    # Xdebugが接続するホストを指定する
xdebug.client_port = 9003    # Xdebugが接続するポート番号を指定する
xdebug.log = /tmp/xdebug.log    # Xdebugのログのパスを指定する
```
`zend_extension` について、分からない場合はDockerコンテナ内で手作業でXdebugのインストールコマンド実行し、パスを確認します。    
`xdebug.client_host` について、Dockerコンテナで実行しているXdebugが接続するホストはDockerを実行しているホストなので、それを指定します。  
`xdebug.client_port` について、任意のポート番号を指定できますが、デフォルトが9003なのでそれを指定します。  
`xdebug.log` について、任意のパスを指定します。

`Dockerfile` にXdebugをインストールするコマンドを追加します。  
今回はDockerイメージに `amazonlinux` を使用しています。  

`Dockerfile` :
```dockerfile
FROM amazonlinux:2023

～省略～

# Xdebugインストール
RUN yum -y install php-pear php-devel
RUN pecl install xdebug-3.3.1
COPY php/conf.d/99-xdebug.ini /etc/php.d/99-xdebug.ini    # 設定ファイルをコピー

～省略～
```
設定ファイルのコピー元の `php/conf.d/99-xdebug.ini` は、`Dockerfile` があるディレクトリから見た `99-xdebug.ini` の相対パスです。  
作成した `99-xdebug.ini` のパスを指定してください。

Xdebugをインストールできたか確認します。  
XdebugをインストールしたDockerコンテナに接続し、以下を実行します。  
```
# php -i | grep xdebug
```
Xdebugの設定がいろいろ表示されたらインストール成功です。

## 3. VSCodeの拡張機能であるPHP Debugをインストールする
VSCodeの拡張機能であるPHP Debugをインストールします。  
VSCodeを開き、「拡張機能」をクリックします。  
![拡張機能を開くボタン](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "拡張機能を開くボタン")

検索バーに「php debug」と入力し、検索結果の「PHP Debug」をインストールします。  
![拡張機能のPHP Debug](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image2.png "拡張機能のPHP Debug")

設定ファイルを作成します。  
「実行とデバッグ」をクリックします。  
![実行とデバッグを開くボタン](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image3.png "実行とデバッグを開くボタン")

「launch.jsonファイルを作成します」をクリックします。  
![launch.jsonファイルを作成するリンク](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image4.png "launch.jsonファイルを作成するリンク")

デバッガーの選択で「PHP」を選択します。  
![デバッガーの選択](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image5.png "デバッガーの選択")

作成された `launch.json` に `pathMappings` を追加します。  
VSCodeで開いたディレクトリにDockerコンテナ内の対応するディレクトリをマップします。  
今回はワークスペースのルートディレクトリにDockerコンテナ内のマウントされたディレクトリをマップしています。
```json
{
    ～省略～
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Listen for Xdebug",
            "type": "php",
            "request": "launch",
            "port": 9003,
            // pathMappings追加
            "pathMappings": {
                "/srv/example.com": "${workspaceFolder}"
            }
        },
        ～省略～
    ]
}
```
PHP Debugのインストールと設定ができました。

## 4. Xdebugを確認する
ブレークポイントで処理が止まるかどうか試します。  

Dockerコンテナを起動します。  

VSCodeで、 `launch.json` で設定した「Listen for Xdebug」を実行します。  
![デバッグの開始ボタン](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image6.png "デバッグの開始ボタン")

ブラウザからアクセスできるPHPファイルを開き、適当な行にブレークポイントを設定します。  
![ブレークポイント](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image7.png "ブレークポイント")

ブラウザからPHPファイルにアクセスします。    
![ブレークポイントで止まる処理](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image8.png "ブレークポイントで止まる処理")

ブレークポイントで処理が止まることを確認できました。
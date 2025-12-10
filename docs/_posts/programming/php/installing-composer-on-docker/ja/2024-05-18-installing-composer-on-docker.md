---
layout: my-post
title: "Dockerイメージビルド時にComposerをインストールする"
date: 2024-05-18 00:00:00 +0000
categories: programming php
page_name: installing-composer-on-docker
lang: ja
image: /assets/images/programming/php/installing-composer-on-docker/image1.png
---

Dockerイメージビルド時にComposerをインストールするようにします。

![サムネイル](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "サムネイル")

## 環境
- Windows 10 64ビット
- Ubuntu 22.04.3 LTS (WSLで起動している)
- Docker Engine 26.0.0
- Amazon Linux 2023(DockerコンテナのOS)

## インストールの流れ
1. [Composerインストール用のスクリプトを作成する。](#1-composerインストール用のスクリプトを作成する)
2. [Dockerイメージビルド時にComposerをインストールする。](#2-dockerイメージビルド時にcomposerをインストールする)
3. [インストールの確認をする。](#3-インストールの確認をする)

## 1. Composerインストール用のスクリプトを作成する
Composerをインストールするためのスクリプトを作成します。  

[このページ](https://getcomposer.org/doc/faqs/how-to-install-composer-programmatically.md)を参考にスクリプトを作ります。  
今回は `install-composer.sh` という名前で保存しています。

`install-composer.sh` :
```bash
#!/bin/bash

set -euxo pipefail

# 期待されるチェックサムを取得
EXPECTED_CHECKSUM="$(php -r 'copy("https://composer.github.io/installer.sig", "php://stdout");')"
# Composerのインストーラーを取得
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
# 実際のチェックサムを生成
ACTUAL_CHECKSUM="$(php -r "echo hash_file('sha384', 'composer-setup.php');")"

# 実際のチェックサムが期待されるものと異なればエラーにする
if [ "$EXPECTED_CHECKSUM" != "$ACTUAL_CHECKSUM" ]
then
    >&2 echo 'ERROR: Invalid installer checksum'
    rm composer-setup.php
    exit 1
fi

# Composerのバージョンを指定できるようにする
# 第2引数でバージョンを指定できるようにする
# 何も指定しなければ最新バージョンになるようにする
VERSION_OPTION=''
if [ $# == 2 ]
then
    VERSION_OPTION="--version=$2"
fi

# Composerをインストールする
# バージョン指定のオプションを付ける
php composer-setup.php $VERSION_OPTION
rm composer-setup.php

# Composerのパスを指定しなくてもComposerを実行できるようにする
# 第1引数でComposerを保存するパスを指定できるようにする
COMPOSER_PATH=$1
mv composer.phar $COMPOSER_PATH
```

## 2. Dockerイメージビルド時にComposerをインストールする
Dockerイメージビルド時にComposerをインストールするようにします。

[上記のスクリプト](#1-composerインストール用のスクリプトを作成する)(`install-composer.sh`)をイメージビルド時に実行するようにします。  
`Dockerfile` を作成します。`install-composer.sh` と同じ場所に作成しています。

`Dockerfile` :
```dockerfile
# 今回はAmazon Linux 2023を使用する
FROM amazonlinux:2023

# Composerのインストールに必要なのでphpとunzipをインストールする
# 任意のバージョンを指定する
RUN yum -y install php8.2 unzip-6.0

# install-composer.shを任意の場所にコピーする
COPY install-composer.sh /tmp/install-composer.sh
# install-composer.shを実行する
# Composerの保存場所とバージョンを引数として指定する(Composerの保存場所はPATHに追加されていればどこでもいい)
RUN /tmp/install-composer.sh /usr/local/bin/composer 2.7.6
```

## 3. インストールの確認をする
Composerがインストールされているか確認します。

`Dockerfile` と同じ場所で以下を実行します。

```bash
$ docker build . -t installed_composer
$ docker run -it --rm --name composer_verification installed_composer bash
```

上記のコマンドで、`installed_composer` という名前のDockerイメージをビルドし、`composer_verification` という名前のDockerコンテナを起動して接続しています。

コンテナ内で以下を実行します。

```
# composer -v
   ______
  / ____/___  ____ ___  ____  ____  ________  _____
 / /   / __ \/ __ `__ \/ __ \/ __ \/ ___/ _ \/ ___/
/ /___/ /_/ / / / / / / /_/ / /_/ (__  )  __/ /
\____/\____/_/ /_/ /_/ .___/\____/____/\___/_/
                    /_/
Composer version 2.7.6 2024-05-04 23:03:15
```

Composerのバージョンが表示され、Composerがインストールされていることを確認できました。
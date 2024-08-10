---
layout: my-post
title: "Dockerイメージビルド時にNode.jsをインストールする"
date: 2024-06-07 00:00:00 +0000
categories: programming javascript
title_eng: installing-node-on-docker
lang: ja
---

Dockerイメージビルド時にNode.jsをインストールするようにします。

## 参考ページ
- [Download Node.js](https://nodejs.org/en/download/package-manager)

## 環境
- Windows 10 64ビット
- Ubuntu 22.04.3 LTS (WSLで起動している)
- Docker Engine 26.0.0
- Amazon Linux 2023(DockerコンテナのOS)

## インストールの流れ
1. [Node.jsインストール用のスクリプトを作成する。](#1-nodejsインストール用のスクリプトを作成する)
2. [Dockerイメージビルド時にNode.jsをインストールする。](#2-dockerイメージビルド時にnodejsをインストールする)
3. [インストールの確認をする。](#3-インストールの確認をする)

## 1. Node.jsインストール用のスクリプトを作成する
Node.jsをインストールするためのスクリプトを作成します。  

nvm(Node Version Manager)を使ってNode.jsをインストールするスクリプトを作成します。  
[このページ](https://nodejs.org/en/download/package-manager)を参考にスクリプトを作ります。  
今回は `install-node.sh` という名前で保存しています。

`install-node.sh` :
```bash
#!/bin/bash

set -euxo pipefail

# nvmのインストーラーを取得してbashコマンドで実行する
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# nvmを読み込む
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

# Node.jsのバージョンを指定できるようにする
# 第1引数でバージョンを指定できるようにする
VERSION=$1
# Node.jsをインストールする
nvm install $VERSION
```

## 2. Dockerイメージビルド時にNode.jsをインストールする
Dockerイメージビルド時にNode.jsをインストールするようにします。

[上記のスクリプト](#1-nodejsインストール用のスクリプトを作成する)(`install-node.sh`)をイメージビルド時に実行するようにします。  
`Dockerfile` を作成します。`install-node.sh` と同じ場所に作成しています。

`Dockerfile` :
```dockerfile
# 今回はAmazon Linux 2023を使用する
FROM amazonlinux:2023

# nvmのインストールに必要なのでtarとgzipをインストールする
# 任意のバージョンを指定する
RUN yum -y install tar-1.34 gzip-1.12

# .bashrcを作成する
# nvmインストール時に自動で.bashrcにnvmを読み込む処理を書いてくれる
# これをすることでbash起動時に自動でnvmが読み込まれるようになる
RUN touch /root/.bashrc
# install-node.shを任意の場所にコピーする
COPY install-node.sh /tmp/install-node.sh
# install-node.shを実行する
# Node.jsのバージョンを引数として指定する
RUN /tmp/install-node.sh 20.14.0
```

## 3. インストールの確認をする
Node.jsがインストールされているか確認します。

`Dockerfile` と同じ場所で以下を実行します。

```bash
$ docker build . -t installed_node
$ docker run -it --rm --name node_verification installed_node bash
```

上記のコマンドで、`installed_node` という名前のDockerイメージをビルドし、`node_verification` という名前のDockerコンテナを起動して接続しています。

コンテナ内で以下を実行します。

```
# node -v
v20.14.0
# npm -v
10.7.0
```

Node.js、npm(Node Package Manager)のバージョンが表示され、Node.jsがインストールされていることを確認できました。
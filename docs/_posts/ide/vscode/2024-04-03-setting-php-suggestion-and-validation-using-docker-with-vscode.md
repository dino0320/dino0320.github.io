---
layout: my-post
title: "VSCodeのPHPの言語候補機能と検証機能をDockerコンテナ内のPHPで行う"
date: 2024-04-03 00:00:00 +0000
categories: ide vscode
title_eng: setting-php-suggestion-and-validation-using-docker-with-vscode
---

Visual Studio Code(VSCode)のPHPの言語候補機能と検証機能をDockerコンテナ内のPHPで行うようにします。

## 環境
- Ubuntu 22.04.3 LTS (WSLで起動している)
- Docker Engine 26.0.0
- PHP 8.2.15 (cli)
- Visual Studio Code 1.87.2

## 前提
- PHPをインストールしているDockerコンテナがある。  
WSL上のDockerコンテナで動くLaravelプロジェクトの構築については[こちら](/application/server/running-laravel-project-on-nginx)をご覧ください。
- Visual Studio Codeをインストールしている。

## 設定の流れ
1. [DockerコンテナでPHPを実行するスクリプトを作成する。](#1-dockerコンテナでphpを実行するスクリプトを作成する)
2. [VSCodeの拡張機能であるPHP Debugをインストールする。](#2-vscodeの拡張機能であるphp-debugをインストールする)
3. [VSCodeのPHPの設定を変更する。](#3-vscodeのphpの設定を変更する)
4. [設定を確認する。](#4-設定を確認する)

## 1. DockerコンテナでPHPを実行するスクリプトを作成する
DockerコンテナでPHPを実行するスクリプトを作成します。  

### Docker Composeの場合
VSCodeで開いているプロジェクト内の `docker-compose.yml` があるディレクトリに `php.sh` を作成します。

`php.sh` :
```bash
#!/bin/bash

docker compose exec <サービス名> php $@
```

### Docker Composeじゃない場合
VSCodeで開いているプロジェクト内の任意のディレクトリに `php.sh` を作成します。

`php.sh` :
```bash
#!/bin/bash

docker exec -it <コンテナ名> php $@
```

`$@` はシェルスクリプトの引数です。`php.sh <引数1> <引数2>` の場合、`<引数1> <引数2>` を表します。これでDockerコンテナ内で任意の引数でPHPを実行できるようになります。

## 2. VSCodeの拡張機能であるPHP Debugをインストールする
VSCodeの拡張機能であるPHP Debugをインストールします。  
VSCodeを開き、「拡張機能」をクリックします。  
![拡張機能を開くボタン](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.title_eng }}/image1.png "拡張機能を開くボタン")

検索バーに「php debug」と入力し、検索結果の「PHP Debug」をインストールします。  
![拡張機能のPHP Debug](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.title_eng }}/image2.png "拡張機能のPHP Debug")

## 3. VSCodeのPHPの設定を変更する
VSCodeのPHPの設定を変更します。  
VSCodeを開き、「設定」をクリックします。  
![設定ボタン](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.title_eng }}/image3.png "設定ボタン")

「ワークスペース」タブを選択し、「拡張機能」の「PHP」をクリックします。  
`php.suggest.basic` と `php.validate.enable` のチェックを外し、組み込みの言語候補機能と検証機能を無効にします。  
![組み込みの言語候補機能と検証機能のチェックボックス](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.title_eng }}/image4.png "組み込みの言語候補機能と検証機能のチェックボックス")

`php.validate.executablePath` の「setting.jsonで編集」をクリックします。  
![setting.jsonで編集のリンク](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.title_eng }}/image5.png "setting.jsonで編集のリンク")

`setting.json` の `php.validate.executablePath` に `php.sh` のパスを指定します。  
今回はワークスペースのルートディレクトリに置いています。

`setting.json` :
```json
{
    "php.suggest.basic": false,
    "php.validate.enable": false,
    "php.validate.executablePath": "${workspaceFolder}/php.sh", // 変更
}
```

再び「設定」を開き、「ワークスペース」タブの「拡張機能」の「PHP Debug」をクリックします。  
`php.debug.executablePath` の「setting.jsonで編集」をクリックします。  
![setting.jsonで編集のリンク](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.title_eng }}/image6.png "setting.jsonで編集のリンク")

`setting.json` の `php.debug.executablePath` に `php.sh` のパスを指定します。
```json
{
    "php.suggest.basic": false,
    "php.validate.enable": false,
    "php.validate.executablePath": "${workspaceFolder}/php.sh",
    "php.debug.executablePath": "${workspaceFolder}/php.sh" // 変更
}
```

## 4. 設定を確認する
言語候補機能と検証機能を確認します。  
VSCodeのプロジェクト内のPHPファイルで試します。

`array` と入力すると、`array` から始まるPHPの関数が候補として表示されます。 
![言語候補機能の確認](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.title_eng }}/image7.png "言語候補機能の確認")

`;` を入力しないとエラーが表示されます。  
![検証機能の確認](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.title_eng }}/image8.png "検証機能の確認")

設定が確認できました。
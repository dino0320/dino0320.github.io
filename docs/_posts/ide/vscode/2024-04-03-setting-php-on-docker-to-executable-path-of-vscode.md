---
layout: my-post
title: "VSCodeのPHPのExecutable PathにDockerコンテナ内のPHPを設定する"
date: 2024-04-03 00:00:00 +0000
categories: ide vscode
title_eng: setting-php-on-docker-to-executable-path-of-vscode
---

Visual Studio Code(VSCode)のPHPのExecutable PathにDockerコンテナ内のPHPを設定します。  
[こちらのページ](https://www.webdeveloperpal.com/2022/02/08/how-to-setup-vscode-with-php-inside-docker/#google_vignette)を参考にさせていただいてます。

## 環境
- Ubuntu 22.04.3 LTS (WSLで起動している)
- Docker Engine 26.0.0
- PHP 8.2.15 (cli)
- Visual Studio Code 1.87.2

## 前提
- PHPをインストールしているDockerコンテナがある。  
WSL上のDockerコンテナで動くLaravelプロジェクトの構築については[こちら](/web-application-framework/laravel/running-laravel-project-on-nginx)をご覧ください。
- Visual Studio Codeをインストールしている。

## 設定の流れ
1. [DockerコンテナでPHPを実行するスクリプトを作成する。](#1-dockerコンテナでphpを実行するスクリプトを作成する)
2. [VSCodeのPHPの設定を変更する。](#2-vscodeのphpの設定を変更する)

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

## 2. VSCodeのPHPの設定を変更する
VSCodeのPHPの設定を変更します。  
VSCodeを開き、「設定」をクリックします。  
![設定ボタン](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.title_eng }}/image1.png "設定ボタン")

「ワークスペース」タブを選択し、「拡張機能」の「PHP」をクリックします。  
`php.validate.executablePath` の「setting.jsonで編集」をクリックします。  
![setting.jsonで編集のリンク](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.title_eng }}/image2.png "setting.jsonで編集のリンク")

`setting.json` の `php.validate.executablePath` に `php.sh` のパスを指定します。  
今回はワークスペースのルートディレクトリに置いています。

`setting.json` :
```json
{
    "php.validate.executablePath": "./php.sh", // 追加
    "php.validate.run": "onType"
}
```

※ `php.validate.run` を「onType」にすると、入力時にPHPの検証が行われ、エラーのときはエラーが表示されるようになります。

Executable PathにDockerコンテナ内のPHPを設定できました。  
Executable Pathに関する警告も出なくなったと思います。
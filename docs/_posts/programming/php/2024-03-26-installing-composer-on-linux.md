---
layout: my-post
title: "Composerをインストールする"
date: 2024-03-26 00:00:00 +0000
categories: programming php
---

Linux環境でComposerをインストールします。

## Composerとは
プロジェクトが使うPHPのライブラリの依存関係を管理するツールです。プロジェクトで使用したいライブラリを、依存するライブラリも含めてインストールしてくれます。  
Composerは `vendor` ディレクトリを作成し、プロジェクトが使うライブラリを `vendor` ディレクトリにインストールします。

## 環境
- Debian GNU/Linux 12 (bookworm)
- PHP 8.2.17 (cli)

## インストールの流れ
1. [Composerをインストールする。](#1-composerをインストールする)
3. [インストールの確認をする。](#2-インストールの確認をする)

## 1. Composerをインストールする
以下を実行することで最新バージョンのComposerをインストールできます。  
`<インストーラーのチェックサム>` は[Composer Public Keys / Checksums](https://composer.github.io/pubkeys.html)の「Installer Checksum」を見て入力します。  
![Composerインストーラーのチェックサム](/assets/images/programming/php/installing-composer-on-linux/image1.png "Composerインストーラーのチェックサム")  
`<現在のディレクトリ>` には現在のディレクトリが表示されます。
```
# php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
# php -r "if (hash_file('sha384', 'composer-setup.php') === '<インストーラーのチェックサム>') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
Installer verified
# php composer-setup.php
All settings correct for using Composer
Downloading...

Composer (version 2.7.2) successfully installed to: <現在のディレクトリ>/composer.phar
Use it: php composer.phar
# php -r "unlink('composer-setup.php');"
# mv composer.phar /usr/local/bin/composer
```
以下は上記の説明です。

`php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"` で現在のディレクトリにComposerのインストーラーをダウンロードします。

`php -r "if (hash_file('sha384', 'composer-setup.php') === '<インストーラーのチェックサム>') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"` でインストーラーを検証します。ダウンロードしたインストーラーをハッシュ化し、`<インストーラーのチェックサム>` と比較します。等しければ正しいインストーラーです。

`php composer-setup.php` でインストーラーを実行し、Composerをインストールします。

`php -r "unlink('composer-setup.php');"` で不要になったインストーラーを削除します。

`mv composer.phar /usr/local/bin/composer` でComposerのパスを指定しなくてもComposerを実行できるようにします。`/usr/local/bin` (移動先)はPATHに追加されていればここじゃなくても大丈夫です。(例えば `/usr/bin` など)  
このコマンドを実行する際、root権限を求められると思います。rootユーザーで実行するか、先頭に `sudo` をつけて実行してください。(`sudo mv composer.phar /usr/local/bin/composer`)

## 2. インストールの確認をする
インストールできたかを確認するために、Composerのバージョンを確認します。  
以下を実行します。
```
# composer -v
   ______
  / ____/___  ____ ___  ____  ____  ________  _____
 / /   / __ \/ __ `__ \/ __ \/ __ \/ ___/ _ \/ ___/
/ /___/ /_/ / / / / / / /_/ / /_/ (__  )  __/ /
\____/\____/_/ /_/ /_/ .___/\____/____/\___/_/
                    /_/
Composer version 2.7.2 2024-03-11 17:12:18
```
インストールの確認ができました。
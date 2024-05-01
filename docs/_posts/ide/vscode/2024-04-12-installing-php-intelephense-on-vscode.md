---
layout: my-post
title: "Visual Studio CodeでPHP Intelephenseをインストールする"
date: 2024-04-12 00:00:00 +0000
categories: ide vscode
title_eng: installing-php-intelephense-on-vscode
---

Visual Studio Code(VSCode)で拡張機能のPHP Intelephenseをインストールします。

## PHP Intelephenseとは
VSCodeのPHP用の拡張機能です。コード補完や定義へのジャンプなどのPHP開発に必要な機能が使えるようになります。

## 環境
- Windows 10 64ビット
- Visual Studio Code 1.87.2
- PHP 8.2.15

## インストールの流れ
1. [PHP Intelephenseをインストールする。](#1-php-intelephenseをインストールする)
2. [基本的なPHP Intelephenseの設定をする。](#2-基本的なphp-intelephenseの設定をする)
3. [追加でPHP Intelephenseの設定をする。](#3-追加でphp-intelephenseの設定をする)

## 1. PHP Intelephenseをインストールする
VSCodeを開き、「拡張機能」をクリックします。  
![拡張機能を開くボタン](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.title_eng }}/image1.png "拡張機能を開くボタン")

検索バーに「php intelephense」と入力し、検索結果の「PHP Intelephense」をインストールします。  
![拡張機能のPHP Intelephense](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.title_eng }}/image2.png "拡張機能のPHP Intelephense")

PHP Intelephenseをインストールできました。

## 2. 基本的なPHP Intelephenseの設定をする
[公式ページ](https://marketplace.visualstudio.com/items?itemName=bmewburn.vscode-intelephense-client)に乗っているクイックスタート通りに設定します。  
やることは以下の2つです。有料機能を使用する場合ライセンスキーの購入・入力も必要ですが、今回はスキップします。  
1. [組み込みのPHP言語機能を無効にする。](#1-組み込みのphp言語機能を無効にする)
2. [非標準のphpファイル拡張子のglobパターンをfiles.associations設定に追加する。](#2-非標準のphpファイル拡張子のglobパターンをfilesassociations設定に追加する)

### 1. 組み込みのPHP言語機能を無効にする
VSCodeの組み込みのPHP言語機能を無効にします。

VSCodeの「拡張機能」をクリックします。  
検索バーに「@builtin php」と入力し、検索結果の「PHP言語機能」をクリックします。  
![組み込みのPHP言語機能](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.title_eng }}/image3.png "組み込みのPHP言語機能")

「無効にする」をクリックします。  
![無効にするボタン](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.title_eng }}/image4.png "無効にするボタン")

組み込みのPHP言語機能を無効にできました。  
「PHPの基本言語サポート」は有効のままで大丈夫です。

### 2. 非標準のphpファイル拡張子のglobパターンをfiles.associations設定に追加する
files.associations設定に非標準のphpファイル拡張子を追加します。  
公式ページの例に沿って `.module` を追加します。

VSCodeで左下の歯車をクリックし、「設定」をクリックします。  
![設定を開くボタン](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.title_eng }}/image5.png "設定を開くボタン")

ワークスペースタブを開いて検索バーに「files.associations」と入力します。  
検索結果の「Files Associations」の「項目を追加」をクリックします。  
![files.associations設定](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.title_eng }}/image6.png "files.associations設定")

「項目」に「*.module」、「値」に「php」と入力し、「OK」をクリックします。  
![module拡張子の追加](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.title_eng }}/image7.png "module拡張子の追加")

files.associations設定に `.module` を追加できました。

## 3. 追加でPHP Intelephenseの設定をする
クイックスタートの他に追加で設定を行います。  
intelephense.environment.phpVersion設定に自分の環境のPHPのバージョンを入力します。  
これにより、PHPバージョンに応じた提案と診断をしてくれるようになります。

VSCodeで設定を開き、ワークスペースタブで検索バーに「intelephense.environment.phpVersion」と入力します。  
検索結果の「Intelephense Environment PHP Version」の値を自分の環境のPHPバージョンに変更します。  
今回はPHPバージョンが8.2なので、「8.2」を入力しています。  
![intelephense.environment.phpVersion設定](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.title_eng }}/image8.png "intelephense.environment.phpVersion設定")

以上でPHP Intelephenseの設定ができました。
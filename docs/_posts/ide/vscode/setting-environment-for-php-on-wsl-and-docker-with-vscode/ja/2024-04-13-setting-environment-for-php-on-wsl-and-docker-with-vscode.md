---
layout: my-post
title: "Visual Studio CodeでWSL + Docker上のPHPの開発環境を整える"
date: 2024-04-13 00:00:00 +0000
categories: ide vscode
page_name: setting-environment-for-php-on-wsl-and-docker-with-vscode
lang: ja
image: /assets/images/ide/vscode/setting-environment-for-php-on-wsl-and-docker-with-vscode/image4.png
---

Visual Studio Code(VSCode)でWSL + Docker上のPHPを開発するための設定を行います。  
個人的な意見になります。
<!--more-->

![サムネイル](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image4.png "サムネイル")

## 環境
- Windows 10 64ビット
- Ubuntu 22.04.3 LTS (WSLで起動している)
- Docker Engine 26.0.0
- PHP 8.2.15
- Visual Studio Code 1.87.2

## 前提
- WSL上のDockerコンテナで動くPHPのコードがある。  
WSL上のDockerコンテナで動くLaravelプロジェクトの構築については[こちら](/web-application-framework/laravel/running-laravel-project-on-nginx)をご覧ください。
- Visual Studio Codeをインストールしている。

## 設定の流れ
1. [WSL上のプロジェクトを開く。](#1-wsl上のプロジェクトを開く)
2. [Executable Pathを設定する。](#2-executable-pathを設定する)
3. [Xdebugを設定する。](#3-xdebugを設定する)
4. [PHP Intelephenseをインストールする。](#4-php-intelephenseをインストールする)
5. [PHP DocBlockerをインストールする。](#5-php-docblockerをインストールする)

## 1. WSL上のプロジェクトを開く
VSCodeでWSL上のプロジェクトを開く方法については[こちら](/ide/vscode/connecting-to-wsl-with-vscode)をご覧ください。

## 2. Executable Pathを設定する
VSCodeのPHPのExecutable PathにDockerコンテナ内のPHPを設定する方法については[こちら](/ide/vscode/setting-php-on-docker-to-executable-path-of-vscode)をご覧ください。

## 3. Xdebugを設定する
VSCodeでXdebugを設定する方法については[こちら](/ide/vscode/xdebug-php-with-vscode-on-wsl-and-docker)をご覧ください。

## 4. PHP Intelephenseをインストールする
VSCodeでPHP Intelephenseをインストールする方法については[こちら](/ide/vscode/installing-php-intelephense-on-vscode)をご覧ください。

## 5. PHP DocBlockerをインストールする
PHPDocを自動生成してほしいので、PHP DocBlockerをインストールします。  

VSCodeを開き、「拡張機能」をクリックします。  
![拡張機能を開くボタン](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "拡張機能を開くボタン")

検索バーに「php doc」と入力し、検索結果の「PHP DocBlocker」をインストールします。  
![拡張機能のPHP DocBlocker](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image2.png "拡張機能のPHP DocBlocker")

PHP DocBlockerをインストールできました。  

PHPの関数などの上で `/**` を入力してEnterを押すと、PHPDocが自動で生成されるようになります。  
![PHPDoc例](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image3.png "PHPDoc例")
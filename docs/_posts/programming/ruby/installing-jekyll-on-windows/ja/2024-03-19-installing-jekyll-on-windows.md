---
layout: my-post
title: "WindowsでJekyllをインストールする"
date: 2024-03-19 00:00:00 +0000
categories: programming ruby
page_name: installing-jekyll-on-windows
lang: ja
image: /assets/images/programming/ruby/installing-jekyll-on-windows/image1.png
---

Windows環境でJekyllをインストールします。

![サムネイル](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "サムネイル")

## Jekyllとは
Rubyで作られた静的サイトジェネレーターです。  
HTMLやMarkdownで書かれたファイルから静的サイトを作成します。  

## 環境
- Windows 10 64ビット
- Ruby 3.2.3

## インストールの流れ
1. [Ruby+DevKitをインストールする。](#1-rubydevkitをインストールする)
2. [Jekyllをインストールする。](#2-jekyllをインストールする)
3. [インストールの確認をする。](#3-インストールの確認をする)

以下のページを参考にしています。  
[Installing Ruby and Jekyll](https://jekyllrb.com/docs/installation/windows/)

## 1. Ruby+DevKitをインストールする
Ruby+DevKitのインストールについては[こちら](/programming/ruby/installing-ruby-on-windows)をご覧ください。   
インストールウィザードのインストール完了画面で、「Run 'ridk install' to set up MSYS2 and development toolchain.MSYS2 is required to install gems with C extensions.」を選択してください。

## 2. Jekyllをインストールする
コマンドプロンプトかPowerShellを開き、以下を実行します。
```
> gem install jekyll bundler
```

## 3. インストールの確認をする
インストールできたかを確認するために、Jekyllのバージョンを確認します。  
コマンドプロンプトかPowerShellを開き、以下を実行します。  
```
> jekyll -v
jekyll 4.3.3
```
インストールの確認ができました。
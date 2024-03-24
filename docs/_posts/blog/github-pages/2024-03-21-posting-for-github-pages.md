---
layout: my-post
title: "GitHub Pagesのサイトに記事を投稿する"
date: 2024-03-21 00:00:00 +0000
categories: blog github-pages
---

GitHub Pagesのサイトに記事を投稿します。  
静的サイトジェネレーターのJekyllを使用します。  

## 環境
- Windows 10 64ビット
- Git for Windows 2.44.0
- Ruby 3.2.3
- Jekyll 4.3.3

## 前提
- Gitをインストールしている。
- GitHubのアカウントを持っている。

## 記事投稿の流れ
1. [Jekyllを使用してGitHub Pagesのサイトを作る。](#1-jekyllを使用してgithub-pagesのサイトを作る)
2. [記事を作成する。](#2-記事を作成する)
3. [ローカルで記事を確認する。](#3-ローカルで記事を確認する)
4. [変更をコミット・プッシュする。](#4-変更をコミットプッシュする)
5. [記事を確認する。](#5-記事を確認する)

## 1. Jekyllを使用してGitHub Pagesのサイトを作る
Jekyllを使用してのGitHub Pagesのサイト作成については[こちら](/blog/github-pages/creating-site-by-github-pages-with-jekyll)をご覧ください。  

## 2. 記事を作成する
HTMLかMarkdownで記事を作成します。以下の3点に気を付けます。  
- `_post` ディレクトリに配置する。  
サイトを作成したディレクトリ以下の `_post` ディレクトリに記事を配置します。ない場合は作成します。
- ファイル名を `年-月-日-タイトル` にする。  
「年」は4桁、「月」、「日」は2桁になるようにします。記事のURLのディレクトリ部分は `/年/月/日/タイトル` のようになります。  
例: `2024-03-21-posting-for-github-pages.md`
- ファイルの先頭にfront matterを追加する。  
front matterはJekyllが処理するために必要なYAML形式のブロックです。ページ用の変数を定義できます。  
例:  
```
---
layout: post
title: "GitHub Pagesのサイトに記事を投稿する"
date: 2024-03-21 00:00:00 +0000
categories: blog github-pages
---
```
上記の例の変数の説明です。  
例の他にも変数がいろいろあります。[こちら](https://jekyllrb.com/docs/front-matter/)をご確認ください。

|変数|説明|
|----|----|
|layout|サイトを作成したディレクトリ以下の `_layout` ディレクトリにあるレイアウト用のファイル名を入力します。`jekyll new` コマンドを実行した場合、デフォルトでは[minima](https://github.com/jekyll/minima)テーマを使用します。そのためminimaが定義しているレイアウトを使うことができます。(post、pageなど)|
|title|記事のタイトルを入力します。|
|date|投稿日時を入力します。|
|categories|記事のカテゴリを入力します。スペースで区切ることで複数指定できます。記事のURLのディレクトリ部分が `/blog/github-pages/2024/03/21/posting-for-github-pages` のようになります。|

front matterの下に記事を書いていきます。  
例: 
```
---
layout: post
title: "GitHub Pagesのサイトに記事を投稿する"
date: 2024-03-21 00:00:00 +0000
categories: blog github-pages
---

GitHub Pagesのサイトに記事を投稿します。  
静的サイトジェネレーターのJekyllを使用します。  
```

## 3. ローカルで記事を確認する
1. ローカルでサーバーを立ち上げる。  
コマンドプロンプトかPowerShellを開き、以下を実行します。
```
> cd <サイトを作成した場所>
> bundle exec jekyll serve
```
2. 記事のURLにアクセスする。  
記事のURLは `http://localhost:4000/年/月/日/タイトル` のようになります。  
front matterで `categories` や `permalink` を定義した場合、変わる可能性があります。

## 4. 変更をコミット・プッシュする
コマンドプロンプトかPowerShellを開き、以下を実行します。
```
> cd <リポジトリの場所>
> git add <サイトを作成した場所>/_post/<記事のファイル名>
> git commit -m "記事を追加"
> git push origin main
```

## 5. 記事を確認する
公開した記事のURLにアクセスします。  
記事のURLは `https://<GitHubのアカウント名>.github.io/年/月/日/タイトル` のようになります。  
front matterで `categories` や `permalink` を定義した場合、変わる可能性があります。
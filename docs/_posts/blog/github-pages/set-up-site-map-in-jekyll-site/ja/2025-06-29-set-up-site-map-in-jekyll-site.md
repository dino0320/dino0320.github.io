---
layout: my-post
title: "JekyllのWebサイトでサイトマップを設定する"
date: 2025-06-29 00:00:00 +0000
categories: blog github-pages
page_name: set-up-site-map-in-jekyll-site
lang: ja
---

Jekyll Sitemap Generator Pluginを使ってJekyllのWebサイトにサイトマップを追加します。

## 参考ページ
- [jekyll-sitemap](https://github.com/jekyll/jekyll-sitemap)

## 環境
- Jekyll 4.3.3

## 設定の流れ
1. [Gemfileを更新する](#1-gemfileを更新する)
2. [設定ファイルを更新する](#2-設定ファイルを更新する)
3. [生成されたサイトマップを確認する](#3-生成されたサイトマップを確認する)

## 1. Gemfileを更新する
`Gemfile` に以下の行を追加します。

```
gem 'jekyll-sitemap'
```

> 補足: GitHub PagesはローカルのGemfileを使用しません。
[Dependency versions](https://pages.github.com/versions/)によると、GitHub Pagesでは `jekyll-sitemap` バージョン1.4.0が使用されています。

## 2. 設定ファイルを更新する
`_config.yml` に以下の設定を追加します。

```yml
url: "https://example.com" # the base hostname & protocol for your site
plugins:
  - jekyll-sitemap
```

## 3. 生成されたサイトマップを確認する
サイトをビルドした後、`https://example.com/sitemap.xml` にアクセスすることでサイトマップを確認できます。`https://example.com` は、あなたのサイトのホスト名とプロトコルに置き換えてください。
---
layout: my-post
title: "JekyllのWebサイトでGoogle Analyticsを設定する"
date: 2025-06-29 00:00:00 +0000
categories: blog github-pages
page_name: set-google-analytics-in-jekyll-site
lang: ja
---

JekyllのWebサイトにGoogle AnalyticsのGoogleタグを設定する方法を解説します。  
ここではJekyllのテーマとしてMinimaを使用しています。

## 参考ページ
- [GitHub PagesでMinimaテンプレートを用いてGoogle Analytics GA4を設定する方法](https://zenn.dev/mato/articles/5c9f8e389e231b)

## 環境
- Jekyll 4.3.3
- Minima 2.5.1

## 設定の流れ
1. [Googleタグ用のJavaScriptを追加する](#1-googleタグ用のjavascriptを追加する)
2. [設定ファイルを更新する](#2-設定ファイルを更新する)

## 1. Googleタグ用のJavaScriptを追加する
`_includes` ディレクトリ内に、以下の内容の `google-analytics.html` ファイルを作成、またはすでにあるファイルを以下の内容に更新します。  
すでにファイルがあってもGoogle Analytics 4に対応してないと思うので、変更が必要です。  
参考にした記事は[こちら](https://zenn.dev/mato/articles/5c9f8e389e231b)です。

{% raw %}
```html
<!-- Google tag (gtag.js) -->
<script async src="https://www.googletagmanager.com/gtag/js?id={{ site.google_analytics }}"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', '{{ site.google_analytics }}');
</script>

```
{% endraw %}

## 2. 設定ファイルを更新する
`_config.yml` ファイルに以下の行を追加し、値をGoogleタグに置き換えてください。  
Googleタグは[こちら](https://marketingplatform.google.com/about/analytics/)から取得できます。  
Googleタグは通常 `G-` から始まります。

```yml
google_analytics: YOUR_GOOGLE_TAG
```

上記のコードを有効にするには、JekyllのWebサイトのビルド時に環境変数として `JEKYLL_ENV=production` を指定する必要があります。
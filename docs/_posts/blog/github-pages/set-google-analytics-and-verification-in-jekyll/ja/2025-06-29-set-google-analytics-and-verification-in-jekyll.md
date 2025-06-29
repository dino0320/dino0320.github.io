---
layout: my-post
title: "JekyllのWebサイトでGoogle AnalyticsとVerificationを設定する"
date: 2025-06-29 00:00:00 +0000
categories: blog github-pages
page_name: set-google-analytics-and-verification-in-jekyll
lang: ja
---

JekyllのWebサイトにGoogle AnalyticsのGoogleタグとWebサイトの所有者の確認用のHTMLメタタグを設定する方法を解説します。
ここではJekyllのテーマとしてMinimaを使用しています。

## 参考ページ
- [GitHub PagesでMinimaテンプレートを用いてGoogle Analytics GA4を設定する方法](https://zenn.dev/mato/articles/5c9f8e389e231b)

## 環境
- Jekyll 4.3.3
- Minima 2.5.1

## 設定の流れ
1. [必要なコードを取得する](#1-必要なコードを取得する)  
2. [Google Analyticsのコードを追加する](#2-google-analyticsのコードを追加する)  
3. [Webサイトの所有者の確認用コードを追加する](#3-webサイトの所有者の確認用コードを追加する)  
4. [設定ファイルを更新する](#4-設定ファイルを更新する)

## 1. 必要なコードを取得する
Google AnalyticsのGoogleタグは[こちら](https://marketingplatform.google.com/about/analytics/)から取得できます。
Webサイトの所有者の確認用のHTMLメタタグは[Google Search Console](https://search.google.com/search-console/about)から取得してください。

## 2. Google Analyticsのコードを追加する
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

## 3. Webサイトの所有者の確認用コードを追加する
`_includes` ディレクトリ内に、以下の内容の `head.html` ファイルを作成、または既にあるファイルを以下のように更新します。

{% raw %}
```html
<head>
  <meta charset="utf-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  {%- seo -%}
  <link rel="stylesheet" href="{{ "/assets/main.css" | relative_url }}">
  {%- feed_meta -%}
  {%- if jekyll.environment == 'production' and site.google_analytics -%}
    {%- include google-analytics.html -%}
  {%- endif -%}
  <!-- 以下を追加 -->
  {%- if jekyll.environment == 'production' and site.google_verification -%}
  <meta name="google-site-verification" content="{{ site.google_verification }}" />
  {%- endif -%}
</head>

```
{% endraw %}

## 4. 設定ファイルを更新する
`_config.yml` ファイルに以下の行を追加し、値を実際に取得したコードに置き換えてください。
Google AnalyticsのGoogleタグは通常 `G-` から始まります。

```yml
google_analytics: YOUR_CODE_FOR_GOOGLE_ANALYTICS
google_verification: YOUR_CODE_FOR_GOOGLE_SITE_VERIFICATION
```

上記のコードを有効にするには、JekyllのWebサイトのビルド時に環境変数として `JEKYLL_ENV=production` を指定する必要があります。
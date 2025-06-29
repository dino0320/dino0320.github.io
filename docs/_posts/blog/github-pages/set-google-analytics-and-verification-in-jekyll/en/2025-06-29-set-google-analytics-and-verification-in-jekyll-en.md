---
layout: my-post
title: "Set Google Analytics and Verification in Jekyll"
date: 2025-06-29 00:00:00 +0000
categories: blog github-pages
page_name: set-google-analytics-and-verification-in-jekyll-en
lang: en
---

This article is is about adding a google tag for Google Analytics and a HTML tag for Google Site Verification to a Jekyll website.  
I'm using the Minima theme for the Jekyll website.

## Reference
- [GitHub PagesでMinimaテンプレートを用いてGoogle Analytics GA4を設定する方法](https://zenn.dev/mato/articles/5c9f8e389e231b)

## Environment
- Jekyll 4.3.3
- Minima 2.5.1

## Workflow to set Google Analytics and Verification
1. [Get the Required Codes](#1-get-the-required-codes)  
2. [Add the Code for Google Analytics](#2-add-the-code-for-google-analytics)  
3. [Add the Code for Google Site Verification](#3-add-the-code-for-google-site-verification)  
4. [Update the Config](#4-update-the-config)

## 1. Get the Required Codes
You can obtain the google tag from [Google Analytics](https://marketingplatform.google.com/about/analytics/).  
For the site verification HTML meta tag, visit [Google Search Console](https://search.google.com/search-console/about).

## 2. Add the Code for Google Analytics
Create or update a `google-analytics.html` file in the `_includes` directory with the following content.  
There may be the default file but it's not compatible with Google Analytics 4. So you need to change it.  
I used [this website](https://zenn.dev/mato/articles/5c9f8e389e231b) as reference.

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

## 3. Add the Code for Google Site Verification
Create or update a `head.html` file in the `_include` directory with the following content.

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
  <!-- Add the following lines -->
  {%- if jekyll.environment == 'production' and site.google_verification -%}
  <meta name="google-site-verification" content="{{ site.google_verification }}" />
  {%- endif -%}
</head>

```
{% endraw %}

## 4. Update the Config
Add the following lines to the `_config.yml` file, replacing the placeholders with the actual codes you obtained.  
The code for Google Analytics starts from `G-`.

```yml
google_analytics: YOUR_CODE_FOR_GOOGLE_ANALYTICS
google_verification: YOUR_CODE_FOR_GOOGLE_SITE_VERIFICATION
```

You need to specify a production environment (`JEKYLL_ENV=production`) in the Jekyll build command to make the codes work.
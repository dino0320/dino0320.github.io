---
layout: my-post
title: "How to Set Up a Sitemap in a Jekyll Site"
date: 2025-06-29 00:00:00 +0000
categories: blog github-pages
page_name: set-up-site-map-in-jekyll-site-en
lang: en
image: /assets/images/blog/github-pages/set-up-site-map-in-jekyll-site-en/image1.png
---

This article is is about adding a site map to a Jekyll website using the Jekyll Sitemap Generator Plugin.

![Thumbnail](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Thumbnail")

## Reference
- [jekyll-sitemap](https://github.com/jekyll/jekyll-sitemap)

## Environment
- Jekyll 4.3.3

## Workflow to Set Up Site Map
1. [Update Gemfile](#1-update-gemfile)  
2. [Update Config](#2-update-config)
3. [Verify Generated Site Map](#3-verify-generated-site-map)

## 1. Update Gemfile
Add the following to your `Gemfile`.  

```
gem 'jekyll-sitemap'
```

> Note: GitHub Pages doesn't use your `Gemfile`. According to [Dependency versions](https://pages.github.com/versions/), it uses `jekyll-sitemap` version 1.4.0.

## 2. Update Config
Add the following to your `_config.yml`.  

```yml
url: "https://example.com" # the base hostname & protocol for your site
plugins:
  - jekyll-sitemap
```

## 3. Verify Generated Site Map
Once the site is built, you can access the generated site map at `https://example.com/sitemap.xml`. Replace `https://example.com` with your hostname and protocol.
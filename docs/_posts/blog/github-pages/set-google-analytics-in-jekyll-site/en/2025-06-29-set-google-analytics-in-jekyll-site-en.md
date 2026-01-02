---
layout: my-post
title: "How to Set Up Google Analytics on a Jekyll Site"
date: 2025-06-29 00:00:00 +0000
categories: blog github-pages
page_name: set-google-analytics-in-jekyll-site-en
lang: en
image: /assets/images/blog/github-pages/set-google-analytics-in-jekyll-site-en/image1.png
---

This article is is about adding a google tag for Google Analytics to a Jekyll website.  
I'm using the Minima theme for the Jekyll website.

![Thumbnail](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Thumbnail")

## Reference
- [GitHub PagesでMinimaテンプレートを用いてGoogle Analytics GA4を設定する方法](https://zenn.dev/mato/articles/5c9f8e389e231b)

## Environment
- Jekyll 4.3.3
- Minima 2.5.1

## Workflow to set Google Analytics and Verification
1. [Add the JavaScript for Google tag](#1-add-the-javascript-for-google-tag)  
2. [Update the Config](#2-update-the-config)

## 1. Add the JavaScript for Google tag
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

## 2. Update the Config
Add the following line to the `_config.yml` file, replacing the placeholder with your google tag.  
You can obtain the google tag from [Google Analytics](https://marketingplatform.google.com/about/analytics/).  
The code for Google Analytics starts from `G-`.

```yml
google_analytics: YOUR_GOOGLE_TAG
```

You need to specify a production environment (`JEKYLL_ENV=production`) in the Jekyll build command to make the code work.
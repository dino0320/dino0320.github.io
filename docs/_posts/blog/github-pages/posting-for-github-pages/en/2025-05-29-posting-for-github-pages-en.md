---
layout: my-post
title: "Posting for GitHub Pages"
date: 2025-05-29 00:00:00 +0000
categories: blog github-pages
page_name: posting-for-github-pages-en
lang: en
image: /assets/images/blog/github-pages/posting-for-github-pages-en/image1.png
---

This article is about posting an article to a GitHub Pages site.  
Use the static site generator Jekyll.

![Thumbnail](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Thumbnail")

## Environment  
- Windows 10 64-bit  
- Git for Windows 2.44.0  
- Ruby 3.2.3  
- Jekyll 4.3.3  

## Prerequisites  
- Git is installed.  
- You have a GitHub account.  

## Workflow for Posting an Article  
1. [Create a GitHub Pages site using Jekyll](#1-create-a-github-pages-site-using-jekyll)  
2. [Create an article](#2-create-an-article)  
3. [Check the article locally](#3-check-the-article-locally)  
4. [Commit and push the changes](#4-commit-and-push-the-changes)  
5. [Check the article](#5-check-the-article)  

## 1. Create a GitHub Pages site using Jekyll  
For details on creating a GitHub Pages site using Jekyll, see [here](/blog/github-pages/creating-site-by-github-pages-with-jekyll-en).

## 2. Create an Article  
Write your article in HTML or Markdown. Pay attention to the following three points:  

- Place the file in the `_posts` directory.  
  Put your article in the `_posts` directory under your site's root directory. Create the directory if it doesn't exist.

- Name the file as `year-month-day-title`.  
  Use four digits for the year, and two digits for the month and day.  
  The article's URL will include `/year/month/day/title`.  
  Example: `2025-05-29-posting-for-github-pages-en.md`

- Add front matter at the top of the file.  
  Front matter is a YAML block required for Jekyll to process the file. You can define variables for the page.  
  Example:
  ```
  ---
  layout: post
  title: "Posting for GitHub Pages"
  date: 2024-03-21 00:00:00 +0000
  categories: blog github-pages
  ---
  ```

Explanation of the variables used above.  
For more variables, refer to [Jekyll front matter documentation](https://jekyllrb.com/docs/front-matter/).

| Variable   | Description |
|------------|-------------|
| layout     | Specify the layout file in the `_layouts` directory. When using the default [minima](https://github.com/jekyll/minima) theme created by `jekyll new`, you can use predefined layouts like `post`, `page`, etc. |
| title      | Enter the optional title of the article. It's not necessary to be the same as the file name. |
| date       | Enter the date and time of the post. |
| categories | Specify the categories. Separate multiple categories with spaces. The article's URL will look like `/blog/github-pages/2025/05/29/posting-for-github-pages-en`. |

Write the content of your article below the front matter.  
Example:
```
---
layout: post
title: "Posting for GitHub Pages"
date: 2025-05-29 00:00:00 +0000
categories: blog github-pages
---

This article is about posting an article to a GitHub Pages site.  
Use the static site generator Jekyll.
```

## 3. Check the Article Locally  
1. Start a local server.  
Open Command Prompt or PowerShell and run:
```
> cd <path to the site directory>
> bundle exec jekyll serve
```

2. Access the article's URL.  
The article will be available at `http://localhost:4000/year/month/day/title`.  
If you defined `categories` or `permalink` in the front matter, the URL may differ.

## 4. Commit and Push the Changes  
Open Command Prompt or PowerShell and run:
```
> cd <path to the repository>
> git add <site directory>/_posts/<article filename>
> git commit -m "Add new article"
> git push origin main
```

## 5. Check the Article  
Access the published article using its URL.  
The article will be available at:  
`https://<GitHub username>.github.io/year/month/day/title`  
Example: `https://dino0320.github.io/2025/05/29/creating-site-by-github-pages-en`  
Note: If `categories` or `permalink` are defined in the front matter, the URL may differ.
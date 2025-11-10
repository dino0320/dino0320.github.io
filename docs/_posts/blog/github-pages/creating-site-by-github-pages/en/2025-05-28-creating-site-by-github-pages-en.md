---
layout: my-post
title: "Creating website by GitHub Pages"
date: 2025-05-28 00:00:00 +0000
categories: blog github-pages
page_name: creating-site-by-github-pages-en
lang: en
image: /assets/images/blog/github-pages/creating-site-by-github-pages-en/image11.png
---

This article is about a way to publish a Website Using GitHub Pages.

![Thumbnail](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image11.png "Thumbnail")

## What is GitHub Pages?

GitHub Pages is a service that automatically publishes static websites from GitHub repositories.  
You can build a website using files written in HTML or Markdown.  

For instructions on creating a site using Jekyll, please refer to [this article](/blog/github-pages/creating-site-by-github-pages-with-jekyll-en).

## Workflow to Create a Website

1. [Create a GitHub account](#1-create-a-github-account)  
2. [Create a GitHub repository](#2-create-a-github-repository)  
3. [Configure repository settings](#3-configure-repository-settings)  
4. [Check if the site is published](#4-check-if-the-site-is-published)  
5. [Visit your site](#5-visit-your-site)

## 1. Create a GitHub Account

If you don't have a GitHub account, create one by following the steps below:

1. Go to [GitHub](https://github.com/).  
2. Click “[Sign up](https://github.com/signup)".  
   ![GitHub Sign Up Button](/assets/images/blog/github-pages/creating-site-by-github-pages/image1.png "GitHub Sign Up Button")
3. Follow the instructions to create your account.

## 2. Create a GitHub Repository

1. After signing in, go to the [new repository page](https://github.com/new).  
   ![New Repository Button](/assets/images/blog/github-pages/creating-site-by-github-pages/image2.png)
2. Configure the following settings:

    | Item              | Description |
    |-------------------|-------------|
    | Owner             | Choose your GitHub account. |
    | Repository name   | Set the name as `yourusername.github.io` (all lowercase). |
    | Description       | Optional description for the repository. |
    | Repository visibility | Choose “Public" (required for free accounts). |
    | Add a README file | Check this option to add a `README.md`. This file, along with `index.html` or `index.md`, can serve as the homepage. |

3. Click **Create repository**.  
   ![Create Repository Button](/assets/images/blog/github-pages/creating-site-by-github-pages/image3.png)

## 3. Configure Repository Settings

1. Go to the repository you created.  
   ![Repository Page](/assets/images/blog/github-pages/creating-site-by-github-pages/image4.png)
2. Click **Settings** from the top menu.  
   ![Settings Button](/assets/images/blog/github-pages/creating-site-by-github-pages/image5.png)
3. In the left sidebar, click **Pages**.  
   ![Pages Menu](/assets/images/blog/github-pages/creating-site-by-github-pages/image6.png)
4. Under **Build and deployment**, set the following:

    | Item   | Description |
    |--------|-------------|
    | Source | Select “Deploy from a branch". |
    | Branch | Choose the `main` branch and root folder `/`. |

5. Click **Save**.

## 4. Check if the Site is Published

1. Open the **Actions** tab in your repository.  
   ![Actions Tab](/assets/images/blog/github-pages/creating-site-by-github-pages/image9.png)
2. If a green check mark appears next to an action, the deployment has completed successfully.  
   ![Action Status](/assets/images/blog/github-pages/creating-site-by-github-pages/image10.png)

## 5. Visit Your Site

1. In the repository, go to **Settings** → **Pages**.  
   ![Settings](/assets/images/blog/github-pages/creating-site-by-github-pages/image5.png)  
   ![Pages Menu](/assets/images/blog/github-pages/creating-site-by-github-pages/image6.png)
2. Click the **Visit site** button.  
   ![Visit Site Button](/assets/images/blog/github-pages/creating-site-by-github-pages/image7.png)  
   Your homepage (e.g., `README.md`) should now be visible.  
   ![Homepage Screenshot](/assets/images/blog/github-pages/creating-site-by-github-pages/image8.png)

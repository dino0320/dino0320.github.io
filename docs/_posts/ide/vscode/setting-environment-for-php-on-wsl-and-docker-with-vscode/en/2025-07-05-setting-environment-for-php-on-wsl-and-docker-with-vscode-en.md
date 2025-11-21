---
layout: my-post
title: "Setting Up a PHP Development Environment on WSL + Docker Using Visual Studio Code"
date: 2025-07-05 00:00:00 +0000
categories: ide vscode
page_name: setting-environment-for-php-on-wsl-and-docker-with-vscode-en
lang: en
image: /assets/images/ide/vscode/setting-environment-for-php-on-wsl-and-docker-with-vscode-en/image4.png
---

This guide explains how to configure Visual Studio Code (VSCode) for PHP development on WSL + Docker.  
Note: This is based on personal experience.

![Thumbnail](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image4.png "Thumbnail")

## Environment
- Windows 10 64-bit
- Ubuntu 22.04.3 LTS (running on WSL)
- Docker Engine 26.0.0
- PHP 8.2.15
- Visual Studio Code 1.87.2

## Prerequisites
- You already have a PHP project running in a Docker container on WSL.  
For details on setting up a Laravel project on Docker inside WSL, refer to [this guide](/web-application-framework/laravel/running-laravel-project-on-nginx-en).
- Visual Studio Code is installed.

## Setup Steps
1. [Open the Project on WSL](#1-open-the-project-on-wsl)
2. [Set the Executable Path](#2-set-the-executable-path)
3. [Configure Xdebug](#3-configure-xdebug)
4. [Install PHP Intelephense](#4-install-php-intelephense)
5. [Install PHP DocBlocker](#5-install-php-docblocker)

## 1. Open the Project on WSL
For instructions on how to open a project on WSL with VSCode, see [this guide](/ide/vscode/connecting-to-wsl-with-vscode-en).

## 2. Set the Executable Path
To configure VSCode to use PHP from inside a Docker container as the executable path, follow [this guide](/ide/vscode/setting-php-on-docker-to-executable-path-of-vscode-en).

## 3. Configure Xdebug
To set up Xdebug in VSCode for debugging PHP running in Docker on WSL, refer to [this guide](/ide/vscode/xdebug-php-with-vscode-on-wsl-and-docker-en).

## 4. Install PHP Intelephense
To install PHP Intelephense in VSCode, follow the steps in [this guide](/ide/vscode/installing-php-intelephense-on-vscode-en).

## 5. Install PHP DocBlocker
To automatically generate PHPDoc comments, install PHP DocBlocker.

Open VSCode, and click on the Extensions icon.

![Open Extensions Button](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Open Extensions Button")

Type `php doc` in the search bar, and install PHP DocBlocker from the search results.

![PHP DocBlocker Extension](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image2.png "PHP DocBlocker Extension")

PHP DocBlocker is now installed.

You can now generate PHPDoc blocks automatically by typing `/**` above a function and pressing Enter.

![PHPDoc Example](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image3.png "PHPDoc Example")
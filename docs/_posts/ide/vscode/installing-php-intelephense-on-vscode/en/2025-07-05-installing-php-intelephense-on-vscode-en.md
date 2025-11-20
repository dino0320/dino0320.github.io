---
layout: my-post
title: "Installing PHP Intelephense in Visual Studio Code"
date: 2025-07-05 00:00:00 +0000
categories: ide vscode
page_name: installing-php-intelephense-on-vscode-en
lang: en
image: /assets/images/ide/vscode/installing-php-intelephense-on-vscode-en/image9.png
---

This guide explains how to install the PHP Intelephense extension in Visual Studio Code (VSCode).

![Thumbnail](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image9.png "Thumbnail")

## What is PHP Intelephense?
PHP Intelephense is an extension for Visual Studio Code that provides essential PHP development features such as code completion, go to definition, and more.

## Environment
- Windows 10 64-bit
- Visual Studio Code 1.87.2
- PHP 8.2.15

## Installation Steps
1. [Install PHP Intelephense](#1-install-php-intelephense)
2. [Configure Basic PHP Intelephense Settings](#2-configure-basic-php-intelephense-settings)
3. [Apply Additional PHP Intelephense Settings](#3-apply-additional-php-intelephense-settings)

## 1. Install PHP Intelephense
Open Visual Studio Code, and click on the Extensions icon.

![Open Extensions Button](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Open Extensions Button")

Type `php intelephense` in the search bar, and install the "PHP Intelephense" extension from the search results.

![PHP Intelephense Extension](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image2.png "PHP Intelephense Extension")

PHP Intelephense is now installed.

## 2. Configure Basic PHP Intelephense Settings
Follow the [official Quick Start guide](https://marketplace.visualstudio.com/items?itemName=bmewburn.vscode-intelephense-client) to configure the extension.    
Here are the two main steps (license key input is required for premium features, but we'll skip that in this guide):  
1. [Disable the built-in PHP language features](#1-disable-the-built-in-php-language-features)
2. [Add Non-standard PHP File Extensions to files.associations](#2-add-non-standard-php-file-extensions-to-filesassociations)

### 1. Disable the built-in PHP language features
To avoid conflicts, disable the built-in PHP language features in VSCode.

Click on the Extensions icon in VSCode.  
In the search bar, type `@builtin php`, and click on "PHP Language Features" in the search results.

![Built-in PHP Language Features](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image3.png "Built-in PHP Language Features")

Click "Disable".

![Disable Button](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image4.png "Disable Button")

The built-in PHP language features are now disabled.  
> Note: You can leave "PHP Basic Language Support" enabled.

### 2. Add Non-standard PHP File Extensions to files.associations
Add non-standard PHP file extensions (such as `.module`) to the `files.associations` setting.  
We'll follow the official example and add `.module`.

Click the gear icon in the bottom left of VSCode, then click "Settings".

![Open Settings Button](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image5.png "Open Settings Button")

In the Workspace tab, type `files.associations` into the search bar.  
Click "Add Item" under the Files Associations section.

![files.associations Setting](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image6.png "files.associations Setting")

For Item, enter `*.module`.  
For Value, enter `php`.  
Click OK.

![Add .module Extension](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image7.png "Add .module Extension")

Now `.module` files will be recognized as PHP.

## 3. Apply Additional PHP Intelephense Settings
Aside from the Quick Start setup, you can apply additional configuration.

Set your PHP version in the `intelephense.environment.phpVersion` setting.  
This allows Intelephense to tailor suggestions and diagnostics to your version of PHP.

Open Settings in VSCode, and in the Workspace tab, search for `intelephense.environment.phpVersion`.  
Set the value to match your PHP version.  
For example, if you're using PHP 8.2, enter `8.2`.

![intelephense.environment.phpVersion Setting](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image8.png "intelephense.environment.phpVersion Setting")

PHP Intelephense is now fully configured.
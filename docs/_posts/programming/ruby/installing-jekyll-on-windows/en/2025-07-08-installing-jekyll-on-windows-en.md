---
layout: my-post
title: "Installing Jekyll on Windows"
date: 2025-07-08 00:00:00 +0000
categories: programming ruby
lang: en
---

This guide explains how to install Jekyll on a Windows system.

## What is Jekyll?
Jekyll is a static site generator written in Ruby.  
It builds static websites from files written in HTML or Markdown. 

## Reference
- [Installing Ruby and Jekyll](https://jekyllrb.com/docs/installation/windows/)

## Environment
- Windows 10 64-bit
- Ruby 3.2.3

## Installation Steps
1. [Install Ruby + DevKit](#1-install-ruby--devkit)
2. [Install Jekyll](#2-install-jekyll)
3. [Verify the Installation](#3-verify-the-installation)

## 1. Install Ruby + DevKit
For instructions on installing Ruby + DevKit, see [this guide](/programming/ruby/installing-ruby-on-windows-en).  
During the final screen of the installation wizard, make sure to select:  
"Run 'ridk install' to set up MSYS2 and development toolchain. MSYS2 is required to install gems with C extensions."

## 2. Install Jekyll
Open Command Prompt or PowerShell and run the following command:

```
> gem install jekyll bundler
```

## 3. Verify the Installation
To confirm that Jekyll was successfully installed, check its version.  
Open Command Prompt or PowerShell and run:

```
> jekyll -v
jekyll 4.3.3
```

If the version is displayed, Jekyll has been installed successfully.
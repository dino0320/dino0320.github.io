---
layout: my-post
title: "Installing WSL"
date: 2025-07-06 00:00:00 +0000
categories: platform windows
page_name: installing-wsl-en
lang: en
image: /assets/images/platform/windows/installing-wsl-en/image3.png
---

This guide explains how to install WSL.

![Thumbnail](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image3.png "Thumbnail")

## What is WSL?
WSL stands for Windows Subsystem for Linux, which allows you to use a Linux environment directly on Windows.

## Environment
- Windows 10 64-bit

## Installation Steps
1. [Install WSL and Ubuntu](#1-install-wsl-and-ubuntu)
2. [Restart your PC](#2-restart-your-pc)
3. [Create a UNIX User](#3-create-a-unix-user)
4. [Verify the Installation](#4-verify-the-installation)

## 1. Install WSL and Ubuntu
Open Command Prompt or PowerShell as administrator and run the following command:  
This installs both the WSL components and Ubuntu.

```
> wsl --install
```

If WSL is already installed and you just want to install Ubuntu, run:

```
> wsl --install -d Ubuntu
```

## 2. Restart your PC
Restart your computer.  
After restarting, Ubuntu will launch automatically—just wait a moment.

![Ubuntu boot screen](/assets/images/platform/windows/installing-wsl-en/image1.png "Ubuntu boot screen")

## 3. Create a UNIX User
After a short wait, you'll be prompted to create a UNIX user.  
Enter a username and password of your choice.

![UNIX user creation screen](/assets/images/platform/windows/installing-wsl-en/image2.png "UNIX user creation screen")  

Once Ubuntu finishes setting up, you can close the window.

## 4. Verify the Installation
To check that everything was installed correctly, run the following command in Command Prompt or PowerShell:

```
> wsl -l
Linux 用 Windows サブシステム ディストリビューション:
Ubuntu (既定)
```

This confirms that WSL and Ubuntu were installed successfully.
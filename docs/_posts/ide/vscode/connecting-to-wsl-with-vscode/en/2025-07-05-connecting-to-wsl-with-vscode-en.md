---
layout: my-post
title: "Connect to WSL with Visual Studio Code"
date: 2025-07-05 00:00:00 +0000
categories: ide vscode
page_name: connecting-to-wsl-with-vscode-en
lang: en
image: /assets/images/ide/vscode/connecting-to-wsl-with-vscode-en/image7.png
---

This guide explains how to connect to WSL (Windows Subsystem for Linux) using Visual Studio Code (VSCode).

![Thumbnail](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image7.png "Thumbnail")

## Environment
- Windows 10 64-bit
- Visual Studio Code 1.87.2

## Prerequisites
- Visual Studio Code is already installed.

## Setup Flow
1. [Install WSL and Ubuntu](#1-install-wsl-and-ubuntu)
2. [Install the WSL Extension for VSCode](#2-install-the-wsl-extension-for-vscode)
3. [Open a Remote Window](#3-open-a-remote-window)

## 1. Install WSL and Ubuntu
For instructions on installing WSL and Ubuntu, please refer to [this guide](/platform/windows/installing-wsl-en).

## 2. Install the WSL Extension for VSCode
Open Visual Studio Code and click on the Extensions icon.  
![Open Extensions Button](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Open Extensions Button")

In the search bar, type "wsl", and install the extension named "WSL" from the results.  
![WSL Extension](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image2.png "WSL Extension")

## 3. Open a Remote Window
Click the Remote Window icon in the bottom-left corner of VSCode.    
![Remote Window Button](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image3.png "Remote Window Button")

A list of options will appear at the top of VSCode. Click "Connect to WSL".  
![Remote Connection Option](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image4.png "Remote Connection Option")

You will now be connected to the default WSL distribution, which is Ubuntu.

To open a folder on the Windows side while connected to WSL, click the "Open Folder" button.  
![Open Folder Button](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image5.png "Open Folder Button")

Windows folders are mounted under the `/mnt/` directory in Ubuntu on WSL.  
For example, if the folder you want to open is on the C drive, it will be under `/mnt/c/`.  
Example:  
![Folder Path Input Bar](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image6.png "Folder Path Input Bar")

After entering the folder path, click "OK" to open the folder.
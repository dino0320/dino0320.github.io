---
layout: my-post
title: "Installing Sourcetree on Windows"
date: 2025-07-08 00:00:00 +0000
categories: version-control git
lang: en
---

This guide walks you through installing Sourcetree on a Windows system.

## What is Sourcetree?
Sourcetree is a free Git client with a graphical interface.  
It makes it easier to see file differences and perform Git operations.

## Environment
- Windows 10 64-bit
- Git for Windows 2.44.0

## Prerequisites
- Git must be installed.

## Installation Steps
1. [Download Sourcetree](#1-download-sourcetree)
2. [Install Sourcetree](#2-install-sourcetree)
3. [Verify the Installation](#3-verify-the-installation)

## 1. Download Sourcetree
Visit the [official Sourcetree website](https://www.sourcetreeapp.com/) and click "Download for Windows".

![Download for Windows Button](/assets/images/version-control/git/installing-sourcetree-on-windows-en/image1.png "Download for Windows Button")   

Read the license agreement and privacy policy, check "I agree to the Atlassian Software License Agreement and Privacy Policy.", and click "Download".

![License Agreement Dialog](/assets/images/version-control/git/installing-sourcetree-on-windows-en/image2.png "License Agreement Dialog")

## 2. Install Sourcetree
Run the downloaded installer. In this case, we’ll run `SourceTreeSetup-3.4.17.exe`.

### Bitbucket Cloud Account Registration Screen
You’ll be prompted to sign in with a Bitbucket Cloud account. For now, we’ll skip this by clicking the "Skip" button.

![Bitbucket Cloud Account Registration Screen](/assets/images/version-control/git/installing-sourcetree-on-windows-en/image3.png "Bitbucket Cloud Account Registration Screen")  

### Tool Selection Screen
Select the tools you want to install and click "Next".

![Tool Selection Screen](/assets/images/version-control/git/installing-sourcetree-on-windows-en/image4.png "Tool Selection Screen")

|Item|Description|
|----|----|
|Git|Choose whether to install Git. In this case, Git was already installed, so it's automatically selected.|
|Mercurial|Choose whether to install Mercurial (another version control system). It’s not needed here, so the box is unchecked.|
|Enable automatic line ending handling|Automatically convert line endings from LF to CRLF when checking out files. Checked in this example.|
|Configure Global Ignore|Use a global ignore file (e.g., for IDE-generated files). Unchecked in this example.|

### Preferences Screen
Enter your name and email address to be used for commits, then click "Next".

![Preferences Screen](/assets/images/version-control/git/installing-sourcetree-on-windows-en/image5.png "Preferences Screen")

The installation will begin.  
You may be asked whether to load an existing SSH key. For this guide, click "No".

![Dialog Whether to Load a SSH Key](/assets/images/version-control/git/installing-sourcetree-on-windows-en/image6.png "Dialog Whether to Load a SSH Key")

## 3. Verify the Installation
Once installation is complete, Sourcetree will launch automatically.  
Try adding a repository to confirm everything is working.   
Click the "Add" button.

![Add Repository Button](/assets/images/version-control/git/installing-sourcetree-on-windows-en/image7.png "Add Repository Button")

Enter the path to your repository and click "Add".

![Add Repository Screen](/assets/images/version-control/git/installing-sourcetree-on-windows-en/image8.png "Add Repository Screen")

The repository was successfully added.
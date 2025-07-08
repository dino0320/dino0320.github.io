---
layout: my-post
title: "Installing Ruby on Windows"
date: 2025-07-08 00:00:00 +0000
categories: programming ruby
lang: en
---

This guide explains how to install Ruby on a Windows system.

## Environment
- Windows 10 64-bit

## Installation Steps
1. [Download RubyInstaller](#1-download-rubyinstaller)
2. [Install Ruby](#2-install-ruby)
3. [Verify the Installation](#3-verify-the-installation)

## 1. Download RubyInstaller
Download RubyInstaller from [here](https://rubyinstaller.org/downloads/).  
Choose the WITH DEVKIT version if you need the DevKit, or WITHOUT DEVKIT if not.  
In this guide, we’ll use the recommended WITH DEVKIT version.

![Recommended Version of RubyInstaller WITH DEVKIT](/assets/images/programming/ruby/installing-ruby-on-windows-en/image1.png "Recommended Version of RubyInstaller WITH DEVKIT")

DevKit is the MSYS2 toolchain required for building native C/C++ extensions for Ruby.  
It’s also needed for frameworks like Ruby on Rails.  
Below is an excerpt from the [RubyInstaller for Windows](https://rubyinstaller.org/downloads/) page:

> MSYS2 is required in order to build native C/C++ extensions for Ruby and is necessary for Ruby on Rails. Moreover it allows the download and usage of hundreds of Open Source libraries which Ruby gems often depend on.

## 2. Install Ruby
Run the RubyInstaller you just downloaded.

### License Agreement Screen
On the license agreement screen:

1. Read the license agreement and check "I accept the License".
2. Click "Next".

![License Agreement Screen](/assets/images/programming/ruby/installing-ruby-on-windows-en/image2.png "License Agreement Screen")

### Installation Location and Options Screen
On the installation location and options screen:

1. Set the installation directory.  
Use the default path or click "Browse" to change it.
2. Select options.  
"Add Ruby executables to your PATH" adds Ruby to your system PATH so you can run ruby from the command line without typing the full path.  
"Associate .rb and .rbw files with this Ruby installation" lets you open .rb and .rbw files with Ruby by double-clicking them.
3. Click "Install".

![Installation Location and Options Screen](/assets/images/programming/ruby/installing-ruby-on-windows-en/image3.png "Installation Location and Options Screen")

### Component Selection Screen
Simply click "Next" on the component selection screen.

![Component Selection Screen](/assets/images/programming/ruby/installing-ruby-on-windows-en/image4.png "Component Selection Screen")

### Installation Complete Screen
On the final screen:

1. Choose whether to run "ridk install".  
Select "Run 'ridk install' to set up MSYS2 and development toolchain. MSYS2 is required to install gems with C extensions." to set up the MSYS2 DevKit.
2. Click "Finish".  

If you selected to run "ridk install", a Command Prompt window will open to begin MSYS2 setup.  
You'll be asked which components to install—press Enter to proceed with the default selection.

![Installation Complete Screen](/assets/images/programming/ruby/installing-ruby-on-windows-en/image5.png "Installation Complete Screen")

![MSYS2 setup](/assets/images/programming/ruby/installing-ruby-on-windows-en/image6.png "MSYS2 setup")

## 3. Verify the Installation
To check if Ruby was installed correctly, verify the version.  
Open Command Prompt or PowerShell and run:

```
> ruby -v
ruby 3.2.3 (2024-01-18 revision 52bb2ac0a6) [x64-mingw-ucrt]
```

If the version is displayed, Ruby is successfully installed.
---
layout: my-post
title: "Installing Composer"
date: 2025-07-07 00:00:00 +0000
categories: programming php
lang: en
---

We will install Composer in a Linux environment.

## What is Composer?
Composer is a tool for managing PHP library dependencies used in your project.  
It installs the libraries you want to use, including any dependencies those libraries require.  
Composer creates a `vendor` directory and installs the required libraries into that directory.

## Environment
- Debian GNU/Linux 12 (bookworm)
- PHP 8.2.17、PHP 8.3.4

## Installation Steps
1. [Install Composer](#1-install-composer)
3. [Verify the Installation](#2-verify-the-installation)

## 1. Install Composer
Run the following commands to install the latest version of Composer.  
Replace `<installer checksum>` with the "Installer Checksum" from the [Composer Public Keys / Checksums page](https://composer.github.io/pubkeys.html).  
`<current directory>` will show your current working directory.

```
# php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
# php -r "if (hash_file('sha384', 'composer-setup.php') === '<installer checksum>') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
Installer verified
# php composer-setup.php
All settings correct for using Composer
Downloading...

Composer (version 2.7.2) successfully installed to: <current directory>/composer.phar
Use it: php composer.phar
# php -r "unlink('composer-setup.php');"
# mv composer.phar /usr/local/bin/composer
```

Here's what each command does:

`php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"` downloads the Composer installer to your current directory.

`php -r "if (hash_file('sha384', 'composer-setup.php') === '<installer checksum>') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"` verifies the installer by comparing its hash with the expected checksum. If it matches, the installer is valid.

`php composer-setup.php` runs the installer to install Composer.

`php -r "unlink('composer-setup.php');"` deletes the installer, as it's no longer needed.

`mv composer.phar /usr/local/bin/composer` moves the Composer executable to a directory included in your system's PATH, so you can use the `composer` command without specifying the full path.  
You may need root privileges for this command. Run it as the root user or prepend `sudo` like this:  
`sudo mv composer.phar /usr/local/bin/composer`  
You can use a different directory, such as `/usr/bin`, if it's included in your PATH.

## 2. Verify the Installation
To confirm that Composer was successfully installed, check its version by running:

```
# composer -v
   ______
  / ____/___  ____ ___  ____  ____  ________  _____
 / /   / __ \/ __ `__ \/ __ \/ __ \/ ___/ _ \/ ___/
/ /___/ /_/ / / / / / / /_/ / /_/ (__  )  __/ /
\____/\____/_/ /_/ /_/ .___/\____/____/\___/_/
                    /_/
Composer version 2.7.2 2024-03-11 17:12:18
```

Composer has been successfully installed.
---
layout: my-post
title: "Setting the PHP Executable Path in VSCode to Use PHP Inside a Docker Container"
date: 2025-07-05 00:00:00 +0000
categories: ide vscode
page_name: setting-php-on-docker-to-executable-path-of-vscode-en
lang: en
---

This guide explains how to configure Visual Studio Code (VSCode) to use PHP inside a Docker container as the executable path.
Reference: [How to setup VSCode with PHP inside docker](https://www.webdeveloperpal.com/2022/02/08/how-to-setup-vscode-with-php-inside-docker/#google_vignette)

## Environment
- Ubuntu 22.04.3 LTS (running on WSL)
- Docker Engine 26.0.0
- PHP 8.2.15
- Visual Studio Code 1.87.2

## Prerequisites
- You have a Docker container that has PHP installed.
For instructions on setting up a Laravel project running on Docker in WSL, see [this guide](/web-application-framework/laravel/running-laravel-project-on-nginx-en).
- Visual Studio Code is installed.

## Setup Steps
1. [Create a Script to Run PHP Inside the Docker Container](#1-create-a-script-to-run-php-inside-the-docker-container)
2. [Update the PHP Settings in VSCode](#2-update-the-php-settings-in-vscode)

## 1. Create a Script to Run PHP Inside the Docker Container
First, create a script that allows you to run PHP inside your Docker container.

### If Using Docker Compose
Create a file named `php.sh` in the same directory where your `docker-compose.yml` is located within your project.

`php.sh`:
```bash
#!/bin/bash

docker compose exec <service-name> php $@
```

### If Not Using Docker Compose
Create a `php.sh` file in any directory within your project.

`php.sh` :
```bash
#!/bin/bash

docker exec -it <container-name> php $@
```

`$@` represents all the arguments passed to the shell script.  
For example, running `php.sh <arg1> <arg2>` will forward `<arg1> <arg2>` to the php command inside the container.
This allows you to run PHP inside Docker with any arguments.

## 2. Update the PHP Settings in VSCode
Now update VSCode the PHP Settings in VSCode.

Open VSCode and click on Settings.

![Settings Button](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Settings Button")

Switch to the "Workspace" tab, go to "Extensions" → "PHP", and click "Edit in settings.json" in `php.validate.executablePath`.

![Edit in settings.json link](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image2.png "Edit in settings.json link")

Add the path to `php.sh` to the `php.validate.executablePath` field.  
In this example, the script is placed in the root directory of the workspace.

`setting.json` :
```json
{
    "php.validate.executablePath": "./php.sh", // Added
    "php.validate.run": "onType"
}
```

> Setting "php.validate.run" to "onType" enables real-time validation of your PHP code as you type, displaying errors immediately.

Now your PHP executable path in VSCode points to PHP inside the Docker container.  
You should also stop seeing warnings related to the PHP executable path.
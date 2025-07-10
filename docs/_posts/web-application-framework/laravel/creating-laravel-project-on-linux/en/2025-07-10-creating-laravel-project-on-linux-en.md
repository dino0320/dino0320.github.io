---
layout: my-post
title: "Creating a Laravel Project"
date: 2025-07-10 00:00:00 +0000
categories: web-application-framework laravel
page_name: creating-laravel-project-on-linux-en
lang: en
---

This guide explains how to create a Laravel project in a Linux environment.  
We’ll install Laravel using Docker and create a new project.

## What is Laravel?
Laravel is one of the web application frameworks for PHP.  
A web application framework is a structured environment that makes it easier to develop web apps.  
Laravel is a server-side framework.

## Environment
- Ubuntu 22.04.3 LTS (running on WSL)
- Docker Engine 26.0.0
- Composer 2.7.2

## Project Creation Flow
1. [Start a PHP Container](#1-start-a-php-container)
2. [Install Composer](#2-install-composer)
3. [Install Laravel and Create the Project](#3-install-laravel-and-create-the-project)
4. [Verify the Laravel Project](#4-verify-the-laravel-project)

## 1. Start a PHP Container
First, start a Docker container to install Laravel.  
Run the following commands:
`<path-to-laravel-project-dir>` refers to the directory where you want to create your Laravel project.

```bash
$ cd <path-to-laravel-project-dir>
$ docker run -it --rm --name php_to_install_laravel -w /app -v `pwd`:/app php:8.3 bash
```

The command ``docker run -it --rm --name php_to_install_laravel -w /app -v `pwd`:/app php:8.3 bash`` starts a container named `php_to_install_laravel` from the `php:8.3` image and connects to it using `bash`.  
The container name is optional.
(See more on [docker run command](/platform/docker/about-docker-commands#docker-run-en))

## 2. Install Composer
Composer is required to install Laravel.  
Install Composer inside the connected container.  
For details, refer to [this guide](/programming/php/installing-composer-on-linux-en).

## 3. Install Laravel and Create the Project
Install Laravel (version 11) and create your project.  
Inside the container, run:

```
# composer create-project laravel/laravel:^11.0 <project-name>
```

Replace `<project-name>` with your desired Laravel project name.

If `unzip` or `7z` is not installed, the Laravel installation may fail.  
In that case, install `unzip` with the following commands and try again:

```
# apt update
# apt install unzip
# composer create-project laravel/laravel:^11.0 <project-name> # Retry Laravel installation
```

If successful, the Laravel project should be created in the current directory.  
To confirm, run:

```
# ls
<project-name>
```

Since the `docker run` command used the `-v` option to mount the host directory into the container, the project is also visible from the host system.  
You can check from the host like this:

```
# exit # Exit the container
$ ls
<project-name>
```

## 4. Verify the Laravel Project
Now, check the Laravel project locally.  
Start a Docker container to run the Laravel server:

```bash
$ cd <parent-directory-of-laravel-project>
$ docker run -it --rm --name php_to_confirm_laravel -w /app -v `pwd`:/app -p 8000:8000 php:8.3 bash
```

The command ``docker run -it --rm --name php_to_confirm_laravel -w /app -v `pwd`:/app -p 8000:8000 php:8.3 bash`` starts a container named `php_to_confirm_laravel` from the `php:8.3` image and connects via `bash`.  
The `-p` option connects the host and container ports.

Inside the container, run the following:

```
# cd <project-name>
# php artisan serve --host 0.0.0.0
```

This command starts a local server.  
Using `--host 0.0.0.0` allows access to the server from outside the container.

Since this container is running in Ubuntu on WSL, you can access the project from Windows via `http://localhost:8000`.

![Laravel Project Screen](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Laravel Project Screen")

You’ve successfully confirmed that the Laravel project is working.
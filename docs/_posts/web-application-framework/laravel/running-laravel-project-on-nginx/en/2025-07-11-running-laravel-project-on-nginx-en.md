---
layout: my-post
title: "Running a Laravel Project with NGINX"
date: 2025-07-11 00:00:00 +0000
categories: web-application-framework laravel
page_name: running-laravel-project-on-nginx-en
lang: en
---

We’ll set up an environment to run a Laravel project using Docker.  
The project will run on NGINX + PHP-FPM.

## What is NGINX?
NGINX is open-source web server software.  
Installing NGINX on a computer enables it to function as a web server.

## Environment
- Ubuntu 22.04.3 LTS (running on WSL)
- Docker Engine 26.0.0
- Composer 2.7.2
- Laravel 11

## Setup Steps
1. [Create a Laravel Project](#1-create-a-laravel-project)
2. [Create Configuration Files](#2-create-configuration-files)
3. [Create Docker Configuration Files](#3-create-docker-configuration-files)
4. [Add permissions](#4-add-permissions)
5. [Check the Laravel Project](#5-check-the-laravel-project)

## 1. Create a Laravel Project
To create a Laravel project, please refer to [this guide](/web-application-framework/laravel/creating-laravel-project-on-linux-en).

## 2. Create Configuration Files
Create the following configuration files within your Laravel project directory.  
In this explanation, the root directory of the Laravel project is referred to as `/`.

### nginx.repo
This configuration file sets up a YUM repository for Amazon Linux 2023.  
Based on [this guide](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#installing-prebuilt-amazon-linux-packages).
Create this file in the `docker/web/nginx` directory.

`docker/web/nginx/nginx.repo` :
```
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/amzn/2023/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/amzn/2023/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
```

### default.conf
This is the NGINX configuration file for the Laravel project, based on [this official documentation](https://laravel.com/docs/11.x/deployment#nginx).  
We only change it as setting `server_name` to `localhost` as this is for testing.  
Create this file in the `docker/web/nginx/conf.d` directory.

`docker/web/nginx/conf.d/default.conf` :
```conf
server {
    listen 80;
    listen [::]:80;
    server_name localhost;
    root /srv/example.com/public;
 
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";
 
    index index.php;
 
    charset utf-8;
 
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
 
    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }
 
    error_page 404 /index.php;
 
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }
 
    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

### zzz-www.conf
This is an additional PHP-FPM configuration file used to override the `.sock` file path.  
Create this file in the `docker/web/php/php-fpm.d` directory.

`docker/web/php/php-fpm.d/zzz-www.conf` :
```conf
[www]
listen = /var/run/php/php8.2-fpm.sock
```

## 3. Create Docker Configuration Files
Create the following Docker-related configuration files under the Laravel project directory.  
The root directory of the Laravel project is assumed to be `/`.

### docker-compose.yml
This is the Docker Compose configuration file.  
Create this file in the project root directory.

`/docker-compose.yml` :
```yml
services:
  web:
    build: ./docker/web
    volumes:
      - .:/srv/example.com
    ports:
      - "8080:80"
    command: bash -c "/srv/example.com/docker/web/start.sh"
```

### Dockerfile
This is the Dockerfile for the web service (NGINX + PHP-FPM).  
We’re using the `amazonlinux` base image with future AWS deployment in mind.  
Create this file in the `docker/web` directory.

`docker/web/Dockerfile` :
```dockerfile
FROM amazonlinux:2023

# Install NGINX
RUN yum -y install yum-utils
COPY nginx/nginx.repo /etc/yum.repos.d/nginx.repo
RUN yum -y install nginx-1.24.0
COPY nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf

# Install php-fpm and php-mysqlnd
RUN yum -y install php8.2-fpm php8.2-mysqlnd
# Not creating the directory in advance will cause an error
RUN mkdir /run/php-fpm
RUN mkdir /var/run/php
COPY php/php-fpm.d/zzz-www.conf /etc/php-fpm.d/zzz-www.conf
```

### start.sh
Create `start.sh` which runs when the Docker container starts.  
Create this file in the `docker/web` directory.

`docker/web/start.sh`:

```sh
#!/bin/bash

set -euxo pipefail

PROJECT_PATH=/srv/example.com

# Start php-fpm and NGINX
# By using -g "daemon off;", NGINX runs in the foreground, preventing the container from exiting automatically.
php-fpm
nginx -g "daemon off;"
```

## 4. Add permissions
Add permissions to certain files so that specific users, such as the Nginx user and root, can access them.

```
$ cd <path-to-your-laravel-project-root>
$ chmod 777 storage/logs
$ chmod 777 storage/framework/views
$ chmod 777 database
$ chmod 777 database/database.sqlite
$ chmod 755 docker/web/start.sh
```

Because `chmod 777` is not recommended, use this setting only in a development environment.

## 5. Check the Laravel Project
Start the Docker container and check the Laravel project.  
Run the following commands:

```bash
$ cd <path-to-your-laravel-project-root>
$ docker compose up --build -d
```

Since the container is running on Ubuntu via WSL, access the app from Windows at: `http://localhost:8080`

![Laravel Project Screen](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Laravel Project Screen")

The Laravel project is now up and running.
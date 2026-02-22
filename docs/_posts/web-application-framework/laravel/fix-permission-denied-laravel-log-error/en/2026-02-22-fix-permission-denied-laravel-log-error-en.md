---
layout: my-post
title: "How to Fix “Permission denied: laravel.log” Error (Laravel)"
date: 2026-02-22 00:00:00 +0000
categories: web-application-framework laravel
page_name: fix-permission-denied-laravel-log-error-en
lang: en
image: /assets/images/web-application-framework/laravel/fix-permission-denied-laravel-log-error-en/image1.png
---

This article explains how to resolve the error:  
**"The stream or file "laravel.log" could not be opened in append mode: Failed to open stream: Permission denied."**

![Thumbnail](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Thumbnail")

## Prerequisite
- You have a Laravel project.  
For running a Laravel project with NGINX, refer to "[Running a Laravel Project with NGINX](/web-application-framework/laravel/running-laravel-project-on-nginx-en)".

## Environment
- Windows 11
- Ubuntu 24.04.3 LTS (WSL2 distribution)
- Docker Engine 28.4.0
- Amazon Linux 2023(OS of the Docker container)
- PHP 8.2.15 (fpm-fcgi)
- NGINX 1.24.0
- Laravel 11, 12

## Fix Permission
Check the permissions of the `laravel.log` file's directory.  
You can find the directory path in the logging configuration.  
In this example, the directory is `storage/logs`.

```
drwxr-xr-x root root logs
```

Currently, only the `root` user can write log files.  
To allow the `nginx` user to write them, change the owner of the directory to `nginx`:

```bash
$ chown -R nginx:nginx storage
```

Verify the updated permissions:

```
drwxr-xr-x nginx nginx logs
```

Then check whether the error has been resolved.

## Change PHP-FPM User
If the error persists, check the PHP-FPM user in the `www.conf` file, usually located in the `/etc/php-fpm.d` directory.

If the user is different from `nginx` (e.g., `apache`), change it to `nginx` and restart PHP-FPM:  

```conf
[www]
user = nginx
group = nginx
```
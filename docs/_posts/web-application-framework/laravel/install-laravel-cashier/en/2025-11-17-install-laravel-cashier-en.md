---
layout: my-post
title: "Install Laravel Cashier (Stripe)"
date: 2025-11-17 00:00:00 +0000
categories: web-application-framework laravel
page_name: install-laravel-cashier-en
lang: en
image: /assets/images/web-application-framework/laravel/use-laravel-cashier-en/image1.png
---

I wrote this article about how to install Laravel Cashier (Stripe), because I got stuck while trying to install it.

![Thumbnail](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Thumbnail")

## Reference
- [Laravel Cashier (Stripe)](https://laravel.com/docs/12.x/billing)
- [PHP bcmath extension](https://www.php.net/manual/en/intro.bc.php)

## Prerequisite
- You have a Laravel project.  
To learn how to run a Laravel project with NGINX, please refer to [this guide](/web-application-framework/laravel/running-laravel-project-on-nginx-en).  
If you are using React, refer to [this article](/web-application-framework/laravel/set-up-laravel-with-react-project-en).

## Environment
- Windows 11
- Ubuntu 24.04.3 LTS (WSL2 distribution)
- Docker Engine 28.4.0
- Amazon Linux 2023(OS of the Docker container)
- PHP 8.2.29 (fpm-fcgi)
- NGINX 1.24.0
- Laravel 12.38.1

To install the Cashier package for Stripe, run the following command using Composer:

```bash
composer require laravel/cashier
```

However, I saw errors showing that there are many candidate versions, like this:

```
  Problem 1
    - Root composer.json requires laravel/cashier * -> satisfiable by laravel/cashier[v1.0.0, v2.0.0, ..., v2.0.8, v3.0.0, ..., v3.0.5, v4.0.0, v4.0.1, v4.0.2, v4.0.3, v5.0.0, ..., v5.0.15, v6.0.0, ..., v6.0.20, v7.0.0, ..., v7.2.2, v8.0.0, v8.0.1, v9.0.0, ..., v9.3.6, v10.0.0, ..., v10.7.1, v11.0.0, ..., v11.3.1, v12.0.0, ..., v12.17.2, v13.0.0, ..., v13.17.0, v14.0.0, ..., v14.14.0, v15.0.0, ..., v15.7.1, v16.0.0, ..., v16.0.5].
    ...
```

So I tried specifying the version and running it again.  
I used `^16.0` because this is the latest version compatible with Laravel 12 apparently.

```bash
composer require laravel/cashier:^16.0
```

The errors decreased:

```
  Problem 1
    - Root composer.json requires laravel/cashier ^16.0 -> satisfiable by laravel/cashier[v16.0.0, ..., v16.0.5].
    - laravel/cashier[v16.0.0, ..., v16.0.5] require moneyphp/money ^4.0 -> satisfiable by moneyphp/money[v4.0.0, ..., v4.8.0].
    - moneyphp/money[v4.0.0, ..., v4.8.0] require ext-bcmath * -> it is missing from your system. Install or enable PHP's bcmath extension.
```

It told me that I needed to install or enable PHP's `bcmath` extension.  
This extension provides arbitrary precision mathematics.  
Since I'm using Docker to build the Laravel project, I added the following command to the `Dockerfile`:

```Dockerfile
FROM amazonlinux:2023

...

RUN yum -y install php8.2-bcmath # Added

...

```

Then I rebuilt the image and ran the container.  
You can verify that `bcmath` is enabled by running:

```bash
> php -i | grep bcmath
Additional .ini files parsed => /etc/php.d/20-bcmath.ini,
bcmath
bcmath.scale => 0 => 0
```

Finally, I was able to install Laravel Cashier using the following command:

```bash
composer require laravel/cashier:^16.0
```
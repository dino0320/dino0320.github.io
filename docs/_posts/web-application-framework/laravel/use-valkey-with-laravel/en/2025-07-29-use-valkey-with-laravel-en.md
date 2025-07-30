---
layout: my-post
title: "Use Valkey with Laravel"
date: 2025-07-29 00:00:00 +0000
categories: web-application-framework laravel
page_name: use-valkey-on-laravel-en
lang: en
---

This article explains how to configure Laravel to use Valkey.

## Reference
- [Client Libraries](https://valkey.io/clients/)

## Environment
- Ubuntu 22.04.3 LTS (running on WSL)
- Docker Engine 26.0.0
- Amazon Linux 2023(Docker container OS)
- PHP 8.2.15 (fpm-fcgi)
- NGINX 1.24.0
- Laravel 11

## Prerequisites
- A Docker container running a Laravel project (using Docker Compose).  
For instructions on running a Laravel project with NGINX, refer to [this guide](/web-application-framework/laravel/running-laravel-project-on-nginx-en).

## Configuration Steps
1. [Create a Valkey Docker Container](#1-create-a-valkey-docker-container)
2. [Update Docker configuration](#2-update-docker-configuration)
3. [Update Laravel Configuration](#3-update-laravel-configuration)
4. [Verify the Configuration](#4-verify-the-configuration)

## 1. Create a Valkey Docker Container
Add a Valkey service configuration to your `docker-compose.yml` file:

```yml
services:

...

  # Add the following
  valkey:    # Service name
    image: valkey/valkey:8.1
```

In this example, version `8.1` is specified to match the versions supported by AWS ElastiCache.

## 2. Update Docker configuration
To use PhpRedis with Valkey, install it during the Docker image build process.

Create a PHP configuration file named `50-valkey.ini` in the same directory as your `Dockerfile` to load PhpRedis when PHP starts.

`50-valkey.ini` :

```ini
extension = redis.so
```

Update your `Dockerfile` to install PhpRedis:

`Dockerfile` :

```dockerfile
FROM amazonlinux:2023

...

# Install PhpRedis
RUN yum -y install php-pear-1:1.10.13 php8.2-devel
RUN pecl install redis-6.1.0
COPY 50-valkey.ini /etc/php.d/50-valkey.ini
```

Refer to [this page](https://valkey.io/clients/) for supported PhpRedis versions.

## 3. Update Laravel Configuration
Modify your `.env` file to use Valkey:

`.env` :

```
REDIS_CLIENT=phpredis
REDIS_HOST=valkey
REDIS_PASSWORD=null
REDIS_PORT=6379
```

## 4. Verify the Configuration
Navigate to the directory containing your `docker-compose.yml` file and run the following command to start the Docker containers.

```bash
$ docker compose up -d --build
```

In `routes/web.php` or a similar file, add the following code to set a Redis key:

`web.php` :

```php
use Illuminate\Support\Facades\Redis;

Route::get('/', function () {
    Redis::set('name', 'Test');
});
```

Access `/` in your browser.

Enter the Valkey container:

```bash
$ docker compose exec valkey bash
```

Connect to Valkey:

```
# redis-cli
```

List all keys stored in Valkey:

```
> keys *
1) "<REDIS_PREFIX>_name"
```

`REDIS_PREFIX` is configured in your `.env` file.

Check the stored value:

```
get "<REDIS_PREFIX>_name"
"Test"
```

As shown, the value `Test` was successfully stored in Valkey.
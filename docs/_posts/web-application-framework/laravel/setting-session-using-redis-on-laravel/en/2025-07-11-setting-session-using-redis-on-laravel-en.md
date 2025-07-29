---
layout: my-post
title: "Configuring Laravel to Use Redis for Session Storage"
date: 2025-07-11 00:00:00 +0000
categories: web-application-framework laravel
page_name: setting-session-using-redis-on-laravel-en
lang: en
---

In this guide, we'll configure Laravel to store sessions in Redis.  
The environment is set up using Docker.

## Environment
- Ubuntu 22.04.3 LTS (running on WSL)
- Docker Engine 26.0.0
- Amazon Linux 2023(Docker container OS)
- PHP 8.2.15 (fpm-fcgi)
- NGINX 1.24.0
- Laravel 11

## Prerequisites
- A Docker container running the Laravel project (using Docker Compose).  
For instructions on running a Laravel project with NGINX, refer to [this guide](/web-application-framework/laravel/running-laravel-project-on-nginx-en).

## Configuration Steps
1. [Create a Redis Docker Container](#1-create-a-redis-docker-container)
2. [Modify the Laravel Project's Docker Container Settings](#2-modify-the-laravel-projects-docker-container-settings)
3. [Update Laravel's Configuration](#3-update-laravels-configuration)
4. [Verify the Configuration](#4-verify-the-configuration)

## 1. Create a Redis Docker Container
Add the Redis container configuration to your `docker-compose.yml` file:

```yml
services:

...

  # Add a Redis container configuration
  redis:    # Service name
    image: redis:7.0
```

We're specifying version `7.0` to align with the versions supported by AWS ElastiCache.

## 2. Modify the Laravel Project's Docker Container Settings
Since we'll be using PhpRedis for Redis operations, we'll install it during the Docker image build process.

First, create a PHP configuration file `50-redis.ini` to load PhpRedis on PHP startup.  
We'll create it in the same directory as the `Dockerfile`.

`50-redis.ini` : 
```ini
extension = redis.so
```

Next, update your `Dockerfile` to install PhpRedis:

`Dockerfile` :
```dockerfile
FROM amazonlinux:2023

...

# Install PhpRedis
RUN yum -y install php-pear-1:1.10.13 php8.2-devel
RUN pecl install redis-6.0.2
COPY 50-redis.ini /etc/php.d/50-redis.ini    # Copy the PHP configuration file
```

## 3. Update Laravel's Configuration
Modify the `.env` file to store sessions in Redis:

`.env` :
```
SESSION_DRIVER=redis          # Change the driver to redis
SESSION_LIFETIME=120
SESSION_ENCRYPT=false
SESSION_PATH=/
SESSION_DOMAIN=null
SESSION_CONNECTION=default    # Specify the Redis server to use from config/database.php
```

## 4. Verify the Configuration
To confirm that sessions are stored in Redis:

Navigate to the directory containing `docker-compose.yml` and run the following command to start the Docker container.

```bash
$ docker compose up -d --build
```

In `routes/web.php` or a similar file, add a route to set a session value:

`web.php` : 
```php
Route::get('/', function (Request $request) {
    $request->session()->put('test_key', 'test_value');
});
```

Access `/` in your browser.

Enter the Redis container:

```bash
$ docker compose exec <service_name> bash
```

Connect to Redis:

```
# redis-cli
```

List all keys stored in Redis:

```
> keys *
1) "laravel_database_6NDAsrurvjXJA7ovtIWVgqFpqAMHYlqy9JIP1DeW"
```

Check the session value:

```
> get laravel_database_6NDAsrurvjXJA7ovtIWVgqFpqAMHYlqy9JIP1DeW
"s:212:\"a:4:{s:6:\"_token\";s:40:\"W5bHdoKGrTO3r9xdrnqU4ZeK5muvsDyajjdpve1J\";s:8:\"test_key\";s:10:\"test_value\";s:9:\"_previous\";a:1:{s:3:\"url\";s:21:\"http://localhost:8080\";}s:6:\"_flash\";a:2:{s:3:\"old\";a:0:{}s:3:\"new\";a:0:{}}}\";"
```

As shown, the session value `test_key` is successfully stored in Redis.
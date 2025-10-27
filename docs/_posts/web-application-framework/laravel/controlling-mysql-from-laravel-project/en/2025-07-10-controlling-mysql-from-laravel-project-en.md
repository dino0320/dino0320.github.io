---
layout: my-post
title: "Interacting with MySQL from a Laravel Project"
date: 2025-07-10 00:00:00 +0000
categories: web-application-framework laravel
page_name: controlling-mysql-from-laravel-project-en
lang: en
---

This guide explains how to configure a Laravel project to interact with MySQL.  
We're using Docker to set up the environment.

## Environment
- Ubuntu 22.04.3 LTS (running on WSL)
- Docker Engine 26.0.0
- PHP 8.2.15
- Laravel 11

## Prerequisites
- You already have a Docker container running a Laravel project (using Docker Compose).  
For instructions on how to run a Laravel project with NGINX, refer to [this guide](/web-application-framework/laravel/running-laravel-project-on-nginx-en).

## Setup Overview
1. [Create a MySQL Docker Container](#1-create-a-mysql-docker-container)
2. [Install php8.2-mysqlnd](#2-install-php82-mysqlnd)
3. [Update Laravel Configuration](#3-update-laravel-configuration)
4. [Verify the Configuration](#4-verify-the-configuration)

## 1. Create a MySQL Docker Container
We'll create a Docker container for MySQL.  
Add the MySQL container configuration to your `docker-compose.yml` file:

`docker-compose.yml` :

```yml
services:
...

  # Add MySQL container settings
  db:                              # service name
    image: mysql:8.0.34
    environment:
      MYSQL_ROOT_PASSWORD: root    # root user password
      MYSQL_DATABASE: database     # default database name
```

In this setup, we're using MySQL version `8.0.34` to match Amazon RDS for future deployment on AWS.

## 2. Install php-mysqlnd
To connect to MySQL from PHP, install the `php-mysqlnd` module in your Laravel project's Docker container.  
Edit the `Dockerfile` of your Laravel container as follows:

`Dockerfile`:

```Dockerfile
RUN yum -y install php8.2-mysqlnd
```

## 3. Update Laravel Configuration
Update Laravel’s settings to connect to the MySQL database.  
Edit the environment variables related to the database in your `.env` file as follows:

`.env` :

```
DB_CONNECTION=mysql
DB_HOST=db           # Use the service name of the MySQL container
DB_PORT=3306
DB_DATABASE=database # Default database name
DB_USERNAME=root     # MySQL root user
DB_PASSWORD=root     # Root password
```

## 4. Verify the Configuration
Check if Laravel can interact with the MySQL database.  
Create a new table and insert data from the Laravel project.

Start the Docker containers.  
Navigate to the directory containing your `docker-compose.yml` file and run:

```bash
$ docker compose up -d --build
```

Connect to the container running your Laravel project.  
Replace `<service-name>` with the actual service name of your Laravel container:

```bash
$ docker compose exec <service-name> bash
```

Inside the container, move to the root directory of your Laravel project:

```
# cd <path-to-laravel-project-root>
```

Generate a model and migration for a `tests` table:

```
# php artisan make:model Test -m
```

Add a `name` column to the `tests` table.  
Edit the migration file for the `tests` table in the `database/migrations/` directory like this:

`*_create_tests_table.php` :

```php
Schema::create('tests', function (Blueprint $table) {
    $table->id();
    $table->string('name'); // Added
    $table->timestamps();
});
```

Update `app/Models/Test.php` to define fillable columns for mass assignment:

```php
protected $fillable = [
    'name',
];
```

Run the migration to create the `tests` table in MySQL:

```
# php artisan migrate
```

Add the following code to a PHP file (like `index.php`) that you can access via a browser:

```php
Test::create([
    'name' => 'test',
]);
```

Access the file through a browser to execute the code.

Now, verify the contents of the MySQL database.  
Exit the Laravel container and connect to the MySQL container:

```
# exit                           # Exit the Laravel container
$ docker compose exec db bash    # Connect to the MySQL container
```

Connect to MySQL using the root password (`root`):

```
# mysql -u root -p
```

Select the database:

```
mysql> use database
```

Check the tests table:

```
mysql> select * from tests;
+----+------+---------------------+---------------------+
| id | name | created_at          | updated_at          |
+----+------+---------------------+---------------------+
|  1 | test | 2024-04-09 08:39:10 | 2024-04-09 08:39:10 |
+----+------+---------------------+---------------------+
```

The `id`, `created_at`, and `updated_at` columns are automatically populated.

This confirms that data was successfully inserted into MySQL from your Laravel project.
---
layout: my-post
title: "Interacting with PostgreSQL from a Laravel Project"
date: 2025-10-27 00:00:00 +0000
categories: web-application-framework laravel
page_name: control-postgresql-from-laravel-project-en
lang: en
image: /assets/images/web-application-framework/laravel/control-postgresql-from-laravel-project-en/image1.png
---

This guide explains how to configure a Laravel project to interact with PostgreSQL.  
We're using Docker to set up the environment.

![Thumbnail](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Thumbnail")

## Environment
- Windows 11
- Ubuntu 24.04.3 LTS (WSL2 distribution)
- Docker Engine 28.4.0
- PHP 8.2.29
- Laravel 12

## Prerequisites
- You already have a Docker container running a Laravel project (using Docker Compose).  
For instructions on how to run a Laravel project with NGINX, refer to [this guide](/web-application-framework/laravel/running-laravel-project-on-nginx-en).

## Setup Overview
1. [Create a PostgreSQL Docker Container](#1-create-a-postgresql-docker-container)
2. [Install php-pgsql](#2-install-php-pgsql)
3. [Update Laravel Configuration](#3-update-laravel-configuration)
4. [Verify the Configuration](#4-verify-the-configuration)

## 1. Create a PostgreSQL Docker Container
We'll create a Docker container for PostgreSQL.  
Add the PostgreSQL container configuration to your `docker-compose.yml` file:

`docker-compose.yml` :

```yml
services:
...

  # Add PostgreSQL container settings
  db:                                             # Service name
    image: postgres:17.6
      - ./initdb.d:/docker-entrypoint-initdb.d    # Directory containing SQL files executed at startup
    environment:
      POSTGRES_DB: database                       # Default database name
      POSTGRES_PASSWORD: superuser                # Superuser password
```

In this setup, we're using PostgreSQL version `17.6` to match Amazon RDS for future deployment on AWS.

Create an `initdb.d/init.sql` file in the same directory as your `docker-compose.yml` file:

`init.sql`:

```sql
CREATE USER "user" WITH PASSWORD 'password';
GRANT CREATE ON SCHEMA public TO "user";
```

This file creates a user named `user` with the password `password`.  
This user will be used by your Laravel project.  
Additionally, the `CREATE` privilege on the `public` schema is granted to the user — which is required for running migrations.

## 2. Install php-pgsql
To connect to PostgreSQL from PHP, install the `php-pgsql` module in your Laravel project's Docker container.  
Edit the `Dockerfile` of your Laravel container as follows:

`Dockerfile`:

```Dockerfile
RUN yum -y install php8.2-pgsql
```

## 3. Update Laravel Configuration
Update Laravel’s settings to connect to the PostgreSQL database.  
Edit the environment variables related to the database in your `.env` file as follows:

`.env` :

```
DB_CONNECTION=pgsql
DB_HOST=db           # Use the service name of the PostgreSQL container
DB_PORT=5432
DB_DATABASE=database # Default database name
DB_USERNAME=user     # PostgreSQL user
DB_PASSWORD=password # PostgreSQL user password
```

## 4. Verify the Configuration
Check if Laravel can interact with the PostgreSQL database.  
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

Run the migration to create the `tests` table in PostgreSQL:

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

Now, verify the contents of the PostgreSQL database.  
Exit the Laravel container and connect to the PostgreSQL container:

```
# exit                           # Exit the Laravel container
$ docker compose exec db bash    # Connect to the PostgreSQL container
```

Connect to PostgreSQL using the user password (`password`):

```
# psql -U user -d database
```

Check the tests table:

```
database=> select * from tests;
 id | name | created_at          | updated_at          
----+------+---------------------+---------------------
  1 | test | 2024-04-09 08:39:10 | 2024-04-09 08:39:10 
```

The `id`, `created_at`, and `updated_at` columns are automatically populated.

This confirms that data was successfully inserted into PostgreSQL from your Laravel project.
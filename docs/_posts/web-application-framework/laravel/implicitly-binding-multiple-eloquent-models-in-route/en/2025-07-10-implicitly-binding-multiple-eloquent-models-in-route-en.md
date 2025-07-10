---
layout: my-post
title: "Implicitly Binding Multiple Eloquent Models in Laravel Route Definitions"
date: 2025-07-10 00:00:00 +0000
categories: web-application-framework laravel
page_name: implicitly-binding-multiple-eloquent-models-in-route-en
lang: en
---

In this post, I explored how to implicitly bind multiple Eloquent models in a Laravel route definition.

## References
- [Routing - Laravel 11.x](https://laravel.com/docs/11.x/routing#implicit-binding)
- [Eloquent: Relationships - Laravel 11.x](https://laravel.com/docs/11.x/eloquent-relationships#one-to-many)

## Prerequisites
- Using MySQL as the database for the Laravel project.  
For information on how to interact with MySQL from a Laravel project, refer to [this guide](/web-application-framework/laravel/controlling-mysql-from-laravel-project-en).

## Environment
- Laravel 11
- mysql 8.0.34 for Linux

## Table of Contents
1. [Create Database Tables and Eloquent Models](#1-create-database-tables-and-eloquent-models)
2. [Insert Data into Database Tables](#2-insert-data-into-database-tables)
3. [Define Routes and Verify Functionality](#3-define-routes-and-verify-functionality)

## 1. Create Database Tables and Eloquent Models
Create two database tables and Eloquent models: `users` and `comments`.

The `comments` table is a child of the users table.  
The child table should have a column indicating the parent table's `id`, typically named `<parent_table_name_without_s>_id`.  
In this case, the `comments` table will have a `user_id` column.

Run the following commands to create the Eloquent models and migrations:

```bash
php artisan make:model User -m    # Skip if the 'users' table already exists
php artisan make:model Comment -m
```

Open the migration file `database/migrations/*_create_comments_table.php` and add the `user_id` column in the `up` method:

`*_create_comments_table.php` :
```php
public function up(): void
{
    Schema::create('comments', function (Blueprint $table) {
        $table->id();
        $table->integer('user_id'); // Added
        $table->timestamps();
    });
}
```

In the `app/Models/User.php` file, add a method to retrieve the user's comments. The method name should match the plural form of the child table's name:

`User.php` :
```php
public function comments(): HasMany
{
    return $this->hasMany(Comment::class);
}
```

## 2. Insert Data into Database Tables
Insert data into the `users` and `comments` tables.

For the `comments` table, set the `user_id` column to correspond to the `id` values in the `users` table:

```
mysql> select * from users;
+----+-     -+---------------------+---------------------+
| id |       | created_at          | updated_at          |
+----+- ... -+---------------------+---------------------+
|  1 |       | 2024-05-08 07:23:50 | 2024-05-08 07:23:50 |
|  2 |       | 2024-05-08 07:24:06 | 2024-05-08 07:24:06 |
+----+-     -+---------------------+---------------------+
```

```
mysql> select * from comments;
+----+---------+---------------------+---------------------+
| id | user_id | created_at          | updated_at          |
+----+---------+---------------------+---------------------+
|  1 |       1 | 2024-05-08 07:24:34 | 2024-05-08 07:24:34 |
|  2 |       1 | 2024-05-08 07:24:42 | 2024-05-08 07:24:42 |
|  3 |       2 | 2024-05-08 07:24:49 | 2024-05-08 07:24:49 |
+----+---------+---------------------+---------------------+
```

## 3. Define Routes and Verify Functionality
Define a route that implicitly binds multiple Eloquent models:

For this example, add the following to `routes/web.php`.

`web.php` :
```php
Route::get('/users/{user}/comments/{comment}', function (User $user, Comment $comment) {
    return $comment;
})->scopeBindings();
```

Call the `scopeBindings` method to implicitly bind multiple Eloquent models.
Access `http://localhost:8080/users/1/comments/1` in your browser.

You can get a comment model with `user_id` column is 1 and `id` column is 1.

```
{"id":1,"user_id":1,"created_at":"2024-05-08T07:24:34.000000Z","updated_at":"2024-05-08T07:24:34.000000Z"}
```

If you try to access a comment that doesn't belong to the specified user: `http://localhost:8080/users/1/comments/3`

You will receive a 404 error, indicating that a comment model with `user_id` equal to 1 and `id` equal to 3 does not exist.

### About the scopeBindings Method
Without calling the `scopeBindings` method, Laravel would attempt to bind the comment model independently, without considering its relationship to the user model.  

In the [above example](#2-insert-data-into-database-tables), if you access `/users/1/comments/3` not calling `scopeBindings`, you get a user model with `id` column is 1 and a comment model with `id` column is 3, which means that Laravel doesn't consider their relationship as the user doesn't have the comment with `id` equal to 3.  
Calling `scopeBindings`, Laravel doesn't retrieve those models and displays a 404 error page instead.

However, as mentioned in the [later section](#bonus-binding-eloquent-models-using-columns-other-than-id), when retrieving data using a column other than `id`, there is no need to call the `scopeBindings` method.

## (Bonus) Binding Eloquent Models Using Columns Other Than id
By default, Laravel binds Eloquent models using the `id` column.  
To perform implicit binding using a different column, make the following changes.

Add a `comment_id` column to the `comments` table and use it to retrieve data.  
Modify the `comments` table and its data as shown below:

`*_create_comments_table.php` :
```php
public function up(): void
{
    Schema::create('comments', function (Blueprint $table) {
        $table->id();
        $table->integer('user_id');
        $table->integer('comment_id'); // Added
        $table->timestamps();
    });
}
```

```
mysql> select * from comments;
+----+---------+------------+---------------------+---------------------+
| id | user_id | comment_id | created_at          | updated_at          |
+----+---------+------------+---------------------+---------------------+
|  1 |       1 |          1 | 2024-05-08 07:24:34 | 2024-05-08 07:24:34 |
|  2 |       1 |          2 | 2024-05-08 07:24:42 | 2024-05-08 07:24:42 |
|  3 |       2 |          1 | 2024-05-08 07:24:49 | 2024-05-08 07:24:49 |
+----+---------+------------+---------------------+---------------------+
```

Add the following to `routes/web.php`:

`web.php` :
```php
Route::get('/users/{user}/comments/{comment:comment_id}', function (User $user, Comment $comment) {
    return $comment;
});
```

As mentioned in [About the scopeBindings method](#about-the-scopebindings-method), there is no need to call the `scopeBindings` method when retrieving models using a column other than `id`.

If you access `http://localhost:8080/users/2/comments/1` in your browser, you can retrieve the comment model with `user_id` equal to 2 and `comment_id` equal to 1.

```
{"id":3,"user_id":2,"comment_id":1,"created_at":"2024-05-08T07:24:49.000000Z","updated_at":"2024-05-08T07:24:49.000000Z"}
```
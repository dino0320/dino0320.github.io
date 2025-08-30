---
layout: my-post
title: "Install Bootstrap in a Laravel Project"
date: 2025-08-30 00:00:00 +0000
categories: web-application-framework laravel
page_name: install-bootstrap-in-laravel-project-en
lang: en
---

This article explains how to install Bootstrap in a Laravel project.

## Reference
- [Bootstrap & Vite](https://getbootstrap.com/docs/5.2/getting-started/vite/)

## Environment
- Amazon Linux 2023(OS of the Docker container)
- Laravel 11
- Vite 5.4.19
- Bootstrap 5.3.8
- Sass 1.91.0

## Prerequisites
- Vite is already installed in your Laravel project.  
For more details, see [this article](/web-application-framework/laravel/installing-vite-with-laravel-on-wsl-and-docker-en).

## Installation Steps
1. [Install Bootstrap and Sass](#1-install-bootstrap-and-sass)
2. [Edit vite.config.js](#2-edit-viteconfigjs)
3. [Import Bootstrap](#3-import-bootstrap)
4. [Try Bootstrap](#4-try-bootstrap)

## 1. Install Bootstrap and Sass
Run the following commands to install Bootstrap and Sass.

```bash
npm i --save bootstrap @popperjs/core
npm i --save-dev sass
```

## 2. Edit vite.config.js
Add an alias in the `vite.config.js` file to simplify imports:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import * as path from 'path'; // Added

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/scss/app.scss', 'resources/js/app.js'],
            refresh: true,
        }),
    ],
    server: {
        host: true,
        hmr: {
            host: 'localhost',
        },
    },
    // Add the following
    resolve: {
        alias: {
            '~bootstrap': path.resolve(__dirname, 'node_modules/bootstrap'),
        }
    },
});
```

## 3. Import Bootstrap
Create a `bootstrap.scss` file to import Bootstrap.

```css
@use '~bootstrap/scss/bootstrap';
```

Then edit the `resources/scss/app.scss` file to import `bootstrap.scss`.

```css
@use './bootstrap';
```

## 4. Try Bootstrap
Create a Blade template under `resources/views` named `test.blade.php`.  
Edit it to load Bootstrap with `@vite` and add a button using the `btn btn-primary` classes:

`test.blade.php` :
```php
<!DOCTYPE html>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
  <title></title>
  <meta charset="utf-8" />
  @vite(['resources/scss/bootstrap.scss'])
</head>
<body>
  <button class="btn btn-primary">Primary button</button>
</body>
</html>
```

Add the following route to `routes/web.php` so that accessing `/` displays the test view:

`web.php` :
```php
Route::get('/', function () {
    return view('test');
});
```

Start the development server so SCSS and JavaScript files are updated dynamically.

In the root directory of your Laravel project, run the following command:  

```
npm run dev
```

Access `/` in your browser.  
You should see a button styled with Bootstrap.

![Primary button](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Primary button")
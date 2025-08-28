---
layout: my-post
title: "Specify an Absolute Path in the Path Option of the Artisan Migrate Command"
date: 2025-08-26 00:00:00 +0000
categories: web-application-framework laravel
page_name: specify-absolute-path-in-path-option-of-artisan-migrate-command-en
lang: en
---

Specify an absolute path in the path option of the `artisan migrate` command.

## Environment
- Laravel 11

If you want to specify an absolute path in the `--path` option of the Laravel `artisan migrate` command, add the `--realpath` option and set its value to `true`:

```bash
php artisan migrate --realpath=true --path=<The absolute path of migrations>
```

If you omit the `--realpath` option or set it to `false`, you should specify a relative path instead.
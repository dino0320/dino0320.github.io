---
layout: my-post
title: "artisan migrateコマンドで絶対パスを指定する"
date: 2025-08-26 00:00:00 +0000
categories: web-application-framework laravel
page_name: specify-absolute-path-in-path-option-of-artisan-migrate-command
lang: ja
---

`artisan migrate` コマンドで絶対パスを指定します。

## 環境
- Laravel 11

Laravelの `artisan migrate` コマンドで `--path` オプションに絶対パスを指定したい場合は、`--realpath` オプションを追加して、その値を `true` に設定してください。

```bash
php artisan migrate --realpath=true --path=<Migrationディレクトリの絶対パス>
```

`--realpath` オプションを省略するか `false` に設定した場合は相対パスを指定する必要があります。
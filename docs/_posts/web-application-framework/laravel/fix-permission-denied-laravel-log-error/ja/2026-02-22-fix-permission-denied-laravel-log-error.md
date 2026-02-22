---
layout: my-post
title: "【Laravel】Permission denied: laravel.log を解消する方法"
date: 2026-02-22 00:00:00 +0000
categories: web-application-framework laravel
page_name: fix-permission-denied-laravel-log-error
lang: ja
image: /assets/images/web-application-framework/laravel/fix-permission-denied-laravel-log-error/image1.png
---
  
**"The stream or file "laravel.log" could not be opened in append mode: Failed to open stream: Permission denied."**  
このエラーが出た時の対処方法をまとめました。

![サムネイル](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "サムネイル")

## 前提
- Laravelプロジェクトがある。  
LaravelプロジェクトをNGINXで動かす方法については「[LaravelプロジェクトをNGINXで動かす](/web-application-framework/laravel/running-laravel-project-on-nginx)」をご覧ください。

## 環境
- Windows 11
- Ubuntu 24.04.3 LTS (WSL2ディストリビューション)
- Docker Engine 28.4.0
- Amazon Linux 2023(DockerコンテナのOS)
- PHP 8.2.15 (fpm-fcgi)
- NGINX 1.24.0
- Laravel 11, 12

## パーミッションを修正する
`laravel.log` があるディレクトリの権限を確認してみます。  
ディレクトリのパスは `config/logging.php` で確認できます。  
今回はデフォルトの `storage/logs` とします。

```
drwxr-xr-x root root logs
```

`root` ユーザーがログのディレクトリを所持していることがわかりましたが、ログを書き込むのは `nginx` ユーザーのはずなので所有者を変更します。

```bash
$ chown -R nginx:nginx storage
```

再び権限を確認します。

```
drwxr-xr-x nginx nginx logs
```

これでエラーは解消されるはずです。

## PHP-FPMユーザーを変更する
それでもエラーが残っている場合はPHP-FPMのユーザーが `nginx` ではない可能性があります。  
`www.conf` (`/etc/php-fpm.d` にあるはず)を確認して `user` や `group` が `apache` などだった場合、`nginx` に変更してPHP-FPMを再起動してください。

```conf
[www]
user = nginx
group = nginx
```
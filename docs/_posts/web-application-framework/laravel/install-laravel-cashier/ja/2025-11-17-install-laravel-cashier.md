---
layout: my-post
title: "Laravel Cashier (Stripe) インストールでつまずいた話"
date: 2025-11-17 00:00:00 +0000
categories: web-application-framework laravel
page_name: install-laravel-cashier
lang: ja
image: /assets/images/web-application-framework/laravel/use-laravel-cashier/image1.png
---

Laravel Cashier (Stripe) をインストールする際につまずいたのでそれをまとめようと思います。

![サムネイル](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "サムネイル")

## 参考
- [Laravel Cashier (Stripe)](https://laravel.com/docs/12.x/billing)
- [PHP bcmath extension](https://www.php.net/manual/en/intro.bc.php)

## 前提
- Laravelプロジェクトを作成済み  
LaravelプロジェクトをNGINXで動かす方法については[こちら](/web-application-framework/laravel/running-laravel-project-on-nginx)をご覧ください。  
Reactを使用する場合は[こちら](/web-application-framework/laravel/set-up-laravel-with-react-project)をご覧ください。

## 環境
- Windows 11
- Ubuntu 24.04.3 LTS (WSL2ディストリビューション)
- Docker Engine 28.4.0
- Amazon Linux 2023(DockerコンテナのOS)
- PHP 8.2.29 (fpm-fcgi)
- NGINX 1.24.0
- Laravel 12.38.1

公式ドキュメントの通り、Cashierパッケージをインストールするために以下のようにComposerを使います。

```bash
composer require laravel/cashier
```

しかし、候補となるバージョンが多すぎるようで以下のようなエラーが表示されます。

```
  Problem 1
    - Root composer.json requires laravel/cashier * -> satisfiable by laravel/cashier[v1.0.0, v2.0.0, ..., v2.0.8, v3.0.0, ..., v3.0.5, v4.0.0, v4.0.1, v4.0.2, v4.0.3, v5.0.0, ..., v5.0.15, v6.0.0, ..., v6.0.20, v7.0.0, ..., v7.2.2, v8.0.0, v8.0.1, v9.0.0, ..., v9.3.6, v10.0.0, ..., v10.7.1, v11.0.0, ..., v11.3.1, v12.0.0, ..., v12.17.2, v13.0.0, ..., v13.17.0, v14.0.0, ..., v14.14.0, v15.0.0, ..., v15.7.1, v16.0.0, ..., v16.0.5].
    ...
```

とりあえず最新のバージョンであり、Laravel 12と互換性があるらしい(ソースが見つからなかった)16を指定するべく `^16.0` を指定します。

```bash
composer require laravel/cashier:^16.0
```

だいぶエラーが減りました。

```
  Problem 1
    - Root composer.json requires laravel/cashier ^16.0 -> satisfiable by laravel/cashier[v16.0.0, ..., v16.0.5].
    - laravel/cashier[v16.0.0, ..., v16.0.5] require moneyphp/money ^4.0 -> satisfiable by moneyphp/money[v4.0.0, ..., v4.8.0].
    - moneyphp/money[v4.0.0, ..., v4.8.0] require ext-bcmath * -> it is missing from your system. Install or enable PHP's bcmath extension.
```

これによるとPHPの `bcmath` という任意精度の演算をサポートするための拡張機能が必要なようです。  
Laravelプロジェクトの構築にはDockerを使っているため `Dockerfile` に以下の1行を追加します。

```Dockerfile
FROM amazonlinux:2023

...

RUN yum -y install php8.2-bcmath # 追加

...

```

Dockerイメージをビルドしなおしコンテナを再起動します。  
以下のように `bcmath` が有効かどうか確認できます。

```bash
> php -i | grep bcmath
Additional .ini files parsed => /etc/php.d/20-bcmath.ini,
bcmath
bcmath.scale => 0 => 0
```

これでLaravel Cashierがインストールできるはずです。

```bash
composer require laravel/cashier:^16.0
```
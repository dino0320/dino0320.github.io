---
layout: my-post
title: "MySQL Dockerコンテナ起動時にSQLスクリプトを実行する方法"
date: 2026-01-26 00:00:00 +0000
categories: platform docker
page_name: how-to-run-sql-scripts-when-starting-mysql-docker-container
lang: ja
image: /assets/images/platform/docker/how-to-run-sql-scripts-when-starting-mysql-docker-container/image1.png
---

MySQL Dockerコンテナを起動する際にSQLスクリプトを実行する方法についてまとめました。

![Thumbnail](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Thumbnail")

## 環境
- Ubuntu 24.04.3 LTS (WSL2 distribution)
- Docker Engine 28.4.0
- MySQL 8.0.34

## もくじ
- [Use SQL Files](#use-sql-files)
- [Use Shell Scripts](#use-shell-scripts)

## SQLファイルを使用する
MySQL Dockerコンテナを起動するとき、`docker-entrypoint-initdb.d` ディレクトリ内のSQLファイルが実行されます。

まず以下のような `docker-compose.yml` ファイルを作成します。

`docker-compose.yml`:

```yml
services:
  db:
    image: mysql:8.0.34
    volumes:
    - ./initdb.d:/docker-entrypoint-initdb.d # initdb.d を docker-entrypoint-initdb.d にマウント
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: test_database
```

次に、`docker-compose.yml` と同じ階層に `initdb.d` ディレクトリを作成し、その中に `init.sql` ファイルを作成します。  
これらのSQLはMySQLのrootユーザーとして実行されます。

`initdb.d/init.sql`:

```sql
CREATE TABLE `test` (`id` INTEGER UNSIGNED PRIMARY KEY, `name` VARCHAR(10) NOT NULL);
INSERT INTO `test` (`id`, `name`) VALUES (1, 'Test 1');
INSERT INTO `test` (`id`, `name`) VALUES (2, 'Test 2');
INSERT INTO `test` (`id`, `name`) VALUES (3, 'Test 3');
```

`docker-compose.yml` ファイルと同じディレクトリに移動してコンテナを起動してください。

```bash
docker compose up -d
```

SQLが正しく実行されたか確認します。

```bash
docker compose exec db bash
mysql -u root -p test_database
# rootユーザーのパスワードを入力する (root_password)
mysql> select * from test;
+----+--------+
| id | name   |
+----+--------+
|  1 | Test 1 |
|  2 | Test 2 |
|  3 | Test 3 |
+----+--------+
```

## シェルスクリプトを使用する
SQLのほかに、`docker-entrypoint-initdb.d` ディレクトリ内のシェルスクリプトも実行させることができます。

まず以下のような `docker-compose.yml` ファイルを作成します。

`docker-compose.yml`:

```yml
services:
  db:
    image: mysql:8.0.34
    volumes:
    - ./initdb.d:/docker-entrypoint-initdb.d # initdb.d を docker-entrypoint-initdb.d にマウント
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: test_database
      MYSQL_USER: user # 一般ユーザー
      MYSQL_PASSWORD: password # 一般ユーザーのパスワード
```

次に、`initdb.d` ディレクトリ内に以下のような `init.sh` ファイルを作成します。  
このスクリプトはMySQLユーザーの `user` が `test_database` データベースに接続して、指定したSQLを実行するものです。

`initdb.d/init.sh`:

```sh
#!/bin/bash

set -exo pipefail

mysql -u $MYSQL_USER -p"$MYSQL_PASSWORD" $MYSQL_DATABASE<<-EOSQL
    CREATE TABLE \`test\` (\`id\` INTEGER UNSIGNED PRIMARY KEY, \`name\` VARCHAR(10) NOT NULL);
    INSERT INTO \`test\` (\`id\`, \`name\`) VALUES (1, 'Test 1');
    INSERT INTO \`test\` (\`id\`, \`name\`) VALUES (2, 'Test 2');
    INSERT INTO \`test\` (\`id\`, \`name\`) VALUES (3, 'Test 3');
EOSQL
```

`docker-compose.yml` ファイルと同じディレクトリに移動してコンテナを起動してください。

```bash
docker compose up -d
```

SQLが正しく実行されたか確認します。

```bash
docker compose exec db bash
mysql -u user -p test_database
# userのパスワードを入力する (password)
mysql> select * from test;
+----+--------+
| id | name   |
+----+--------+
|  1 | Test 1 |
|  2 | Test 2 |
|  3 | Test 3 |
+----+--------+
```
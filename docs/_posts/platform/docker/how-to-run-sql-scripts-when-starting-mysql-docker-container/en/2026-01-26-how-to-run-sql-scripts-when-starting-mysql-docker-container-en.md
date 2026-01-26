---
layout: my-post
title: "How to Run SQL Scripts When Starting a MySQL Docker Container"
date: 2026-01-26 00:00:00 +0000
categories: platform docker
page_name: how-to-run-sql-scripts-when-starting-mysql-docker-container-en
lang: en
image: /assets/images/platform/docker/how-to-run-sql-scripts-when-starting-mysql-docker-container-en/image1.png
---

This article explains how to run SQL scripts when starting a MySQL Docker container.

![Thumbnail](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Thumbnail")

## Environment
- Ubuntu 24.04.3 LTS (WSL2 distribution)
- Docker Engine 28.4.0
- MySQL 8.0.34

## Table of Contents
- [Use SQL Files](#use-sql-files)
- [Use Shell Scripts](#use-shell-scripts)

## Use SQL Files
You can run SQL files located in the `docker-entrypoint-initdb.d` directory when starting a MySQL Docker container.

Create a `docker-compose.yml` file as follows:

`docker-compose.yml`:

```yml
services:
  db:
    image: mysql:8.0.34
    volumes:
    - ./initdb.d:/docker-entrypoint-initdb.d # Mount the local initdb.d directory to the container's docker-entrypoint-initdb.d
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: test_database
```

Then create the following `init.sql` file in the `initdb.d` directory located next to the `docker-compose.yml` file.  
These SQL scripts will be executed as the MySQL root user.

`initdb.d/init.sql`:

```sql
CREATE TABLE `test` (`id` INTEGER UNSIGNED PRIMARY KEY, `name` VARCHAR(10) NOT NULL);
INSERT INTO `test` (`id`, `name`) VALUES (1, 'Test 1');
INSERT INTO `test` (`id`, `name`) VALUES (2, 'Test 2');
INSERT INTO `test` (`id`, `name`) VALUES (3, 'Test 3');
```

Navigate to the same directory as the `docker-compose.yml` file and run the following command to start the container:

```bash
docker compose up -d
```

Check if the SQL scripts were executed successfully:

```bash
docker compose exec db bash
mysql -u root -p test_database
# Enter the root password (root_password)
mysql> select * from test;
+----+--------+
| id | name   |
+----+--------+
|  1 | Test 1 |
|  2 | Test 2 |
|  3 | Test 3 |
+----+--------+
```

## Use Shell Scripts
You can also run shell scripts located in the `docker-entrypoint-initdb.d` directory when starting a MySQL Docker container.

Create the `docker-compose.yml` file as follows:

`docker-compose.yml`:

```yml
services:
  db:
    image: mysql:8.0.34
    volumes:
    - ./initdb.d:/docker-entrypoint-initdb.d # Mount the local initdb.d directory to the container's docker-entrypoint-initdb.d
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: test_database
      MYSQL_USER: user # The MySQL username to create
      MYSQL_PASSWORD: password # The password for the MySQL user
```

Then create the following `init.sh` file in the `initdb.d` directory.  
This script connects to the `test_database` database as the MySQL user `user` and executes the SQL statements.

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

Navigate to the same directory as the `docker-compose.yml` file and run the following command to start the container:

```bash
docker compose up -d
```

Check if the SQL scripts were executed successfully:

```bash
docker compose exec db bash
mysql -u user -p test_database
# Enter the user password (password)
mysql> select * from test;
+----+--------+
| id | name   |
+----+--------+
|  1 | Test 1 |
|  2 | Test 2 |
|  3 | Test 3 |
+----+--------+
```
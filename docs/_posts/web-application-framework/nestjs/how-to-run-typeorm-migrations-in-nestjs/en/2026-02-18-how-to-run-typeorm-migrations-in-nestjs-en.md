---
layout: my-post
title: "How to Run TypeORM Migrations in NestJS (MySQL)"
date: 2026-02-18 00:00:00 +0000
categories: web-application-framework nestjs
page_name: how-to-run-typeorm-migrations-in-nestjs-en
lang: en
image: /assets/images/web-application-framework/nestjs/how-to-run-typeorm-migrations-in-nestjs-en/image1.png
---

This article explains how to run TypeORM migrations in a NestJS application.  
In this example, we will use MySQL.

![Thumbnail](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Thumbnail")

## References
- [TypeORM Docs](https://typeorm.io/docs)
- [NestJSでMySQLを使う](https://zenn.dev/bamboohouse/articles/7fc17b44e489b5)
- [TypeORM's migration:generate regenerates the whole database schema](https://stackoverflow.com/questions/52800204/typeorms-migrationgenerate-regenerates-the-whole-database-schema)

## Environment
- Ubuntu 24.04.3 LTS (WSL2 distribution)
- Docker Engine 28.4.0
- Docker Compose 2.39.4
- Amazon Linux 2023(OS of the Docker container)
- Node.js 24.13.1
- NestJS 11.0.1
- NGINX 1.28.2
- MySQL 8.4.7
- @nestjs/typeorm 11.0.0

## Prerequisite
- You have a Docker container running a NestJS project on WSL (using Docker Compose).  
For running a NestJS project with NGINX, refer to "[How to Set Up a NestJS + Nginx Reverse Proxy with Docker Compose](/web-application-framework/nestjs/how-to-set-up-nestjs-and-nginx-reverse-proxy-with-docker-compose-en)".

## Setup Steps
1. [Set Up TypeORM](#1-set-up-typeorm)
2. [Create DataSource](#2-create-datasource)
3. [Create User Resource](#3-create-user-resource)
4. [Create Migration](#4-create-migration)
5. [Run Migrations](#5-run-migrations)

## 1. Set Up TypeORM
Set up TypeORM in your NestJS project.  
Please refer to "[How to Connect NestJS to MySQL Using TypeORM (With Example Implementation)](/web-application-framework/nestjs/how-to-connect-nestjs-to-mysql-using-typeorm-en)".

If you created `database.config.ts` by following the above guide, change the `synchronize` value to `false`.  
This is essential for using migration.

`database.config.ts`:

```ts
import { registerAs } from '@nestjs/config';

export default registerAs('database', () => getDatabaseConfig());

export const getDatabaseConfig = () => ({
  host: process.env.DATABASE_HOST,
  port: process.env.DATABASE_PORT === undefined ? undefined : +process.env.DATABASE_PORT,
  username: process.env.DATABASE_USERNAME,
  password: process.env.DATABASE_PASSWORD,
  database: process.env.DATABASE_DATABASE,
  synchronize: false, // Change to false
})
```

## 2. Create DataSource
Create a DataSource to hold your database connection settings.  

`src/app-data-source.ts`:

```ts
import * as dotenv from 'dotenv'
import { DataSource } from 'typeorm'
import { getDatabaseConfig } from './config/database.config';

dotenv.config()

const databaseConfig = getDatabaseConfig()

export const AppDataSource = new DataSource({
    type: 'mysql',
    ...databaseConfig,
    migrations: [__dirname + '/db/migrations/*{.js,.ts}'],
})
```

If you use `process.env` for your database configuration, call `dotenv.config()` before loading the configuration so that `process.env` is properly initialized.

Even if you only use TypeScript files for migrations in your `src` directory, you should include `.js` in the `migrations` option because the files are compiled to JavaScript in the `dist` directory.

## 3. Create User Resource
Create a User resource as test data.  
Please refer to "[Create User Resource](/web-application-framework/nestjs/how-to-connect-nestjs-to-mysql-using-typeorm-en#1-create-user-resource)".

## 4. Create Migration
Run the following command to create a user migration file:

```bash
$ npx typeorm migration:create src/db/migrations/user
```

You should see the `<Timestamp>-user.ts` in the `src/db/migrations` directory.

Add the logic to create and drop the table:

`<Timestamp>-user.ts`:

```ts
import {
    MigrationInterface,
    QueryRunner,
    Table,
} from "typeorm"

export class User1770805968508 implements MigrationInterface {

    public async up(queryRunner: QueryRunner): Promise<void> {
        await queryRunner.createTable(
            new Table({
                name: "user",
                columns: [
                    {
                        name: "id",
                        type: "int",
                        isPrimary: true,
                        isGenerated: true,
                        generationStrategy: "increment",
                    },
                    {
                        name: "name",
                        type: "varchar",
                    },
                ],
            }),
            true,
        )
    }

    public async down(queryRunner: QueryRunner): Promise<void> {
        await queryRunner.dropTable("user", true)
    }

}

```

`isPrimary: true` sets the column as the primary key.

`isGenerated: true` and `generationStrategy: "increment"` enable `auto_increment`.

## 5. Run Migrations
Run the migrations with the following command:

```bash
$ npx typeorm-ts-node-commonjs migration:run -d ./src/app-data-source.ts
```

You should see a `user` table created in MySQL:

```bash
mysql> show columns from user;
+-------+--------------+------+-----+---------+----------------+
| Field | Type         | Null | Key | Default | Extra          |
+-------+--------------+------+-----+---------+----------------+
| id    | int          | NO   | PRI | NULL    | auto_increment |
| name  | varchar(255) | NO   |     | NULL    |                |
+-------+--------------+------+-----+---------+----------------+
```

If you want to revert the changes, run the following command:

```bash
$ npx typeorm-ts-node-commonjs migration:revert -d ./src/app-data-source.ts
```
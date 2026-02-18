---
layout: my-post
title: "NestJSでTypeORMのマイグレーションを実行する方法【MySQL】"
date: 2026-02-18 00:00:00 +0000
categories: web-application-framework nestjs
page_name: how-to-run-typeorm-migrations-in-nestjs
lang: ja
image: /assets/images/web-application-framework/nestjs/how-to-run-typeorm-migrations-in-nestjs/image1.png
---

NestJSアプリケーションでTypeORMのマイグレーションを実行する方法についてまとめました。  
本記事ではMySQLを使用します。

![サムネイル](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "サムネイル")

## 前提
- [TypeORM Docs](https://typeorm.io/docs)
- [NestJSでMySQLを使う](https://zenn.dev/bamboohouse/articles/7fc17b44e489b5)
- [TypeORM's migration:generate regenerates the whole database schema](https://stackoverflow.com/questions/52800204/typeorms-migrationgenerate-regenerates-the-whole-database-schema)

## 環境
- Ubuntu 24.04.3 LTS (WSL2ディストリビューション)
- Docker Engine 28.4.0
- Docker Compose 2.39.4
- Amazon Linux 2023(DockerコンテナのOS)
- Node.js 24.13.1
- NestJS 11.0.1
- NGINX 1.28.2
- MySQL 8.4.7
- @nestjs/typeorm 11.0.0

## 前提
- WSL上でDocker Composeを使用してNestJSプロジェクトのコンテナが起動している。  
NestJSとNGINXの構築方法については「[Docker ComposeでNestJS + Nginxのリバースプロキシ構成を作る方法](/web-application-framework/nestjs/how-to-set-up-nestjs-and-nginx-reverse-proxy-with-docker-compose)」を参照してください。

## セットアップ手順
1. [TypeORMを設定する](#1-typeormを設定する)
2. [DataSourceを作成する](#2-datasourceを作成する)
3. [Userリソースを作成する](#3-userリソースを作成する)
4. [マイグレーションを作成する](#4-マイグレーションを作成する)
5. [マイグレーションを実行する](#5-マイグレーションを実行する)

## 1. TypeORMを設定する
NestJSプロジェクトにTypeORMを設定します。  
「[NestJSでMySQLに接続する方法【TypeORMで実装例付き】](/web-application-framework/nestjs/how-to-connect-nestjs-to-mysql-using-typeorm)」を参照してください。

上記の記事に従って `database.config.ts` を作成した場合、`synchronize` の値を `false` に変更してください。  
これはマイグレーションを使用するために必須です。

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
  synchronize: false, // falseに変更する
})
```

## 2. DataSourceを作成する
データベース接続設定を保持するためのDataSourceを作成します。

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

データベース設定に `process.env` を使用している場合は設定を読み込む前に `dotenv.config()` を呼び出して `process.env` を初期化してください。

`src` ディレクトリでマイグレーションをTypeScriptのみで作成している場合でも、`migrations` オプションには `.js` を含める必要があります。  
ビルド時に `dist` ディレクトリ内でJavaScriptへコンパイルされるためです。

## 3. Userリソースを作成する
テスト用にUserリソースを作成します。  
「[Userリソースを作成する](/web-application-framework/nestjs/how-to-connect-nestjs-to-mysql-using-typeorm#1-userリソースを作成する)」を参照してください。

## 4. マイグレーションを作成する
以下のコマンドを実行して `user` マイグレーションファイルを作成します。

```bash
$ npx typeorm migration:create src/db/migrations/user
```

`src/db/migrations` ディレクトリに `<Timestamp>-user.ts` が生成されます。

テーブルを作成・削除するロジックを追加します。

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

`isPrimary: true` はそのカラムを主キーに設定します。

`isGenerated: true` と `generationStrategy: "increment"` は `auto_increment` を有効にします。

## 5. マイグレーションを実行する
以下のコマンドでマイグレーションを実行します。

```bash
$ npx typeorm-ts-node-commonjs migration:run -d ./src/app-data-source.ts
```

MySQLに `user` テーブルが作成されていることを確認します。

```bash
mysql> show columns from user;
+-------+--------------+------+-----+---------+----------------+
| Field | Type         | Null | Key | Default | Extra          |
+-------+--------------+------+-----+---------+----------------+
| id    | int          | NO   | PRI | NULL    | auto_increment |
| name  | varchar(255) | NO   |     | NULL    |                |
+-------+--------------+------+-----+---------+----------------+
```

変更を元に戻したい場合は以下のコマンドを実行してください。

```bash
$ npx typeorm-ts-node-commonjs migration:revert -d ./src/app-data-source.ts
```
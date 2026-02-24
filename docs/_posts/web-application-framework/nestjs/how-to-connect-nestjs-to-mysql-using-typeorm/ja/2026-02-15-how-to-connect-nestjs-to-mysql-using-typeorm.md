---
layout: my-post
title: "NestJSでMySQLに接続する方法【TypeORMで実装例付き】"
date: 2026-02-15 00:00:00 +0000
categories: web-application-framework nestjs
page_name: how-to-connect-nestjs-to-mysql-using-typeorm
lang: en
image: /assets/images/web-application-framework/nestjs/how-to-connect-nestjs-to-mysql-using-typeorm/image1.png
---

Docker Compose環境でNestJSからMySQLにTypeORMを使用して接続する方法をまとめました。  
各環境ごとに異なる設定を管理するために、`.env` ファイルを使用します。

SequelizeもNestJSで利用可能ですが、今回はTypeORMを使用します。

![サムネイル](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "サムネイル")

## TypeORMとは？
TypeORMはTypeScriptまたはJavaScriptでデータベースを扱うためのORM（Object-Relational Mapping）ライブラリのひとつです。

## 参照
- [Configuration: NestJS](https://docs.nestjs.com/techniques/configuration)
- [Database: NestJS](https://docs.nestjs.com/techniques/database)
- [CRUD generator: NestJS](https://docs.nestjs.com/recipes/crud-generator)
- [NestJSでMySQLを使う](https://zenn.dev/bamboohouse/articles/7fc17b44e489b5)

## 環境
- Ubuntu 24.04.3 LTS (WSL2ディストリビューション)
- Docker Engine 28.4.0
- Docker Compose 2.39.4
- Amazon Linux 2023(DockerコンテナのOS)
- Node.js 24.13.1
- NestJS 11.0.1
- NGINX 1.28.2
- MySQL 8.4.7
- @nestjs/config 4.0.3
- @nestjs/typeorm 11.0.0

## 前提
- WSL上でDocker Composeを使用してNestJSプロジェクトのコンテナが起動している。  
NestJSとNGINXの構築方法については「[Docker ComposeでNestJS + Nginxのリバースプロキシ構成を作る方法](/web-application-framework/nestjs/how-to-set-up-nestjs-and-nginx-reverse-proxy-with-docker-compose)」を参照してください。

## セットアップ手順
1. [MySQLのDockerコンテナを作成する](#1-mysqlのdockerコンテナを作成する)
2. [ConfigとTypeORMをインストールする](#2-configとtypeormをインストールする)
3. [MySQLを設定する](#3-mysqlを設定する)
4. [ConfigModuleを設定する](#4-configmoduleを設定する)
5. [TypeORMを設定する](#5-typeormを設定する)
6. [データベース接続を確認する](#6-データベース接続を確認する)

## 1. MySQLのDockerコンテナを作成する
MySQL用のDockerコンテナを作成します。  
`docker-compose.yml` に以下の設定を追加します。

`docker-compose.yml` :

```yml
services:
...

  # MySQLコンテナ設定を追加
  db:                              # サービス名
    image: mysql:8.4.7
    environment:
      MYSQL_ROOT_PASSWORD: root    # rootユーザーのパスワード
      MYSQL_DATABASE: database     # デフォルトのデータベース名
```

今回は将来的にAWS上のAmazon RDSへデプロイすることを想定し、MySQLバージョン `8.4.7` を使用しています。

## 2. ConfigとTypeORMをインストールする
以下のコマンドを実行してConfigおよびTypeORMパッケージをインストールします。 
Configパッケージは環境変数を管理するために使用します。

```bash
$ npm i --save @nestjs/config
$ npm install --save @nestjs/typeorm typeorm mysql2
```

`package.json` に `@nestjs/config` と `@nestjs/typeorm` が追加されていればインストールは成功です。

## 3. MySQLを設定する
ローカル開発用の環境変数を定義するため、プロジェクトルートに `.env` ファイルを作成します。

`.env`:

```env
DATABASE_HOST=db
DATABASE_PORT=3306
DATABASE_USERNAME=root
DATABASE_PASSWORD=root
DATABASE_DATABASE=database
```

次に、MySQLの設定を定義するため `src/config` に `database.config.ts` を作成します。

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
  synchronize: true,
})
```

TypeORMのマイグレーションなど、NestJSアプリケーション外でもこの設定を使用するためどこからでも参照できるように `getDatabaseConfig` 関数を定義しています。  
NestJSアプリケーション内のみで使用する場合は `registerAs` のみを定義し `app.module.ts` で読み込めば問題ありません。

`synchronize` を `true` に設定すると、エンティティから自動的にデータベーステーブルが生成されるため開発が容易になります。  
本番環境では `false` に設定することを推奨します。

## 4. ConfigModuleを設定する
MySQL設定をグローバルの `process.env` に読み込むため `src/app.module.ts` を以下のように編集します。

`app.module.ts`:

```ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import databaseConfig from './config/database.config';

@Module({
  imports: [
    // 以下を追加する
    ConfigModule.forRoot({
      isGlobal: true,
      load: [databaseConfig],
    }),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}

```

`isGlobal` を `true` に設定することで、`ConfigModule` を各モジュールごとにインポートする必要がなくなります。

## 5. TypeORMを設定する
NestJSアプリケーションからMySQLへ接続するため、`app.module.ts` を以下のように編集します。

`app.module.ts`:

```ts
import { Module } from '@nestjs/common';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { TypeOrmModule } from '@nestjs/typeorm';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import databaseConfig from './config/database.config';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      load: [databaseConfig],
    }),
    // 以下を追加する
    TypeOrmModule.forRootAsync({
      useFactory: (configService: ConfigService) => ({
        type: 'mysql',
        host: configService.get<string>('database.host'),
        port: configService.get<number>('database.port'),
        username: configService.get<string>('database.username'),
        password: configService.get<string>('database.password'),
        database: configService.get<string>('database.database'),
        synchronize: configService.get<boolean>('database.synchronize'),
        autoLoadEntities: true,
      }),
      inject: [ConfigService],
    }),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}

```

前述のとおり、`isGlobal` を `true` に設定していれば `imports: [ConfigModule]` を追加する必要はありません。

## 6. データベース接続を確認する
データベース接続を確認します。

### 1. Userリソースを作成する
テストデータ用にUserリソースを作成します。  
以下のコマンドを実行します。

```bash
$ nest g resource user
```

`REST API` を選択し、CRUDエントリーポイントを生成します。

Nest CLIがインストールされていない場合は以下を実行してください。

```bash
$ npm install -g @nestjs/cli
```

その後、以下のようにファイルを編集します。

`src/user/entities/user.entity.ts`:

```ts
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity()
export class User {
    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;
}

```

`src/user/dto/create-user.dto.ts`:

```ts
export class CreateUserDto {
    name: string;
}

```

`CreateUserDto` はユーザーからのリクエストデータを受け取り、userテーブルへ渡すために使用されます。

`src/user/user.module.ts`:

```ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UserService } from './user.service';
import { UserController } from './user.controller';
import { User } from './entities/user.entity';

@Module({
  imports: [TypeOrmModule.forFeature([User])],
  controllers: [UserController],
  providers: [UserService],
})
export class UserModule {}

```

`src/user/user.service.ts`:

```ts
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';
import { User } from './entities/user.entity';

@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private userRepository: Repository<User>,
  ) {}

  create(createUserDto: CreateUserDto) {
    return this.userRepository.insert(createUserDto);
  }

  findAll(): Promise<User[]> {
    return this.userRepository.find();
  }

  findOne(id: number): Promise<User | null> {
    return this.userRepository.findOneBy({ id });
  }

  update(id: number, updateUserDto: UpdateUserDto) {
    return this.userRepository.update(id, updateUserDto);
  }

  async remove(id: number): Promise<void> {
    await this.userRepository.delete(id);
  }
}

```

`src/user/user.controller.ts`:

```ts
import { Controller, Get, Post, Body, Patch, Param, Delete } from '@nestjs/common';
import { UserService } from './user.service';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';

@Controller('user')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Post()
  create(@Body() createUserDto: CreateUserDto) {
    return this.userService.create(createUserDto);
  }

  @Get()
  findAll() {
    return this.userService.findAll();
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.userService.findOne(+id);
  }

  @Patch(':id')
  update(@Param('id') id: string, @Body() updateUserDto: UpdateUserDto) {
    return this.userService.update(+id, updateUserDto);
  }

  @Delete(':id')
  remove(@Param('id') id: string) {
    return this.userService.remove(+id);
  }
}

```

`UserModule` はすでに `app.module.ts` にインポートされているはずです。

### 2. 接続確認する
NestJSサーバーを起動します。

```bash
$ docker compose exec web bash    # webサービスに接続する
$ cd <node_modulesのパス>
$ npm run start
```

設定が正しいか確認します。  
以下の `curl` コマンドでPOSTおよびGETリクエストを実行します。

```bash
$ curl -X POST -H "Content-Type: application/json" -d '{"name":"Test"}' http://localhost:8080/user
$ curl  http://localhost:8080/user
[{"id":1,"name":"Test"}]
```

`[{"id":1,"name":"Test"}]` が表示されれば設定は成功です。
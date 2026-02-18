---
layout: my-post
title: "How to Connect NestJS to MySQL Using TypeORM (With Example Implementation)"
date: 2026-02-15 00:00:00 +0000
categories: web-application-framework nestjs
page_name: how-to-connect-nestjs-to-mysql-using-typeorm-en
lang: en
image: /assets/images/web-application-framework/nestjs/how-to-connect-nestjs-to-mysql-using-typeorm-en/image1.png
---

This article explains how to connect NestJS to MySQL using TypeORM with Docker Compose.  
We will use a `.env` file to manage different settings for each environment.  

Although Sequelize is also available for NestJS, we will use TypeORM in this article.

![Thumbnail](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Thumbnail")

## What is TypeORM?
TypeORM is the one of ORM (Object–relational mapping) libraries that allows you to work with databases in TypeScript or JavaScript.  

## References
- [Configuration: NestJS](https://docs.nestjs.com/techniques/configuration)
- [Database: NestJS](https://docs.nestjs.com/techniques/database)
- [CRUD generator: NestJS](https://docs.nestjs.com/recipes/crud-generator)
- [NestJSでMySQLを使う](https://zenn.dev/bamboohouse/articles/7fc17b44e489b5)

## Environment
- Ubuntu 24.04.3 LTS (WSL2 distribution)
- Docker Engine 28.4.0
- Docker Compose 2.39.4
- Amazon Linux 2023(OS of the Docker container)
- Node.js 24.13.1
- NestJS 11.0.1
- NGINX 1.28.2
- MySQL 8.4.7
- @nestjs/config 4.0.3
- @nestjs/typeorm 11.0.0

## Prerequisite
- You have a Docker container running a NestJS project on WSL (using Docker Compose).  
For running a NestJS project with NGINX, refer to "[How to Set Up a NestJS + Nginx Reverse Proxy with Docker Compose](/web-application-framework/nestjs/how-to-set-up-nestjs-and-nginx-reverse-proxy-with-docker-compose-en)".

## Setup Steps
1. [Create MySQL Docker Container](#1-create-mysql-docker-container)
2. [Install Config and TypeORM](#2-install-config-and-typeorm)
3. [Configure MySQL](#3-configure-mysql)
4. [Configure ConfigModule](#4-configure-configmodule)
5. [Configure TypeORM](#5-configure-typeorm)
6. [Verify Database Connection](#6-verify-database-connection)

## 1. Create MySQL Docker Container
Create a Docker container for MySQL.  
Add the MySQL container configuration to your `docker-compose.yml` file:

`docker-compose.yml` :

```yml
services:
...

  # Add MySQL container settings
  db:                              # service name
    image: mysql:8.4.7
    environment:
      MYSQL_ROOT_PASSWORD: root    # root user password
      MYSQL_DATABASE: database     # default database name
```

In this setup, we are using MySQL version `8.4.7` to match Amazon RDS for future deployment on AWS.

## 2. Install Config and TypeORM
Install the Config and TypeORM packages by running the following command.  
The Config package is used to manage environment variables.

```bash
$ npm i --save @nestjs/config
$ npm install --save @nestjs/typeorm typeorm mysql2
```

If you can find `@nestjs/config` and `@nestjs/typeorm` in your `package.json`, the installations were successful.

## 3. Configure MySQL
Create a `.env` file in your project root as follows to define environment variables for local development:

`.env`:

```env
DATABASE_HOST=db
DATABASE_PORT=3306
DATABASE_USERNAME=root
DATABASE_PASSWORD=root
DATABASE_DATABASE=database
```

Create `database.config.ts` in the `src/config` directory to define MySQL configuration:

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

Since I use this configuration outside the NestJS application (for example, in TypeORM migrations), I defined a `getDatabaseConfig` function so that the configuration can be accessed from anywhere.  
If you only use it in your NestJS application, it's fine to define only a `registerAs` method to load it in `app.module.ts`.

If `synchronize` is set to `true`, database tables are automatically generated from entities, which makes development easier.  
In a production environment, it is recommended to set it to `false`.

## 4. Configure ConfigModule
To load the MySQL configuration into the global `process.env`.  
Edit `src/app.module.ts`:

`app.module.ts`:

```ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import databaseConfig from './config/database.config';

@Module({
  imports: [
    // Add the following
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

`isGlobal` allows you not to import `ConfigModule` every time you need.

## 5. Configure TypeORM
To connect to MySQL from your NestJS application, edit `app.module.ts` like the following:

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
    // Add the following
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

As mentioned above, you do not need to add `imports: [ConfigModule]` as long as `isGlobal` is set to `true`.

## 6. Verify Database Connection
Now let's verify the database connection.

### 1. Create User Resource
Create a User resource as test data.

Run the following command to generate the User resource:

```bash
$ nest g resource user
```

Choose `REST API` and generate CRUD entry points.

If you haven't installed Nest CLI yet, run the following command:

```bash
$ npm install -g @nestjs/cli
```

Edit files as follows:

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

`CreateUserDto` is used to receive the user's request data and pass it to the `user` table.

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

`UserModule` should be imported in `app.module.ts` already.

### 2. Verify Database Connection
Run your NestJS server:

```bash
$ docker compose exec web bash    # Connect to the web service
$ cd <Your node_modules path>
$ npm run start
```

Check if the settings are proper.  
Run the following `curl` command to POST and GET a request:

```bash
$ curl -X POST -H "Content-Type: application/json" -d '{"name":"Test"}' http://localhost:8080/user
$ curl  http://localhost:8080/user
[{"id":1,"name":"Test"}]
```

If you see `[{"id":1,"name":"Test"}]`, the configuration was successful.
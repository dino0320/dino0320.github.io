---
layout: my-post
title: "Setting Up a PHP CLI Environment with Docker"
date: 2025-07-06 00:00:00 +0000
categories: platform docker
page_name: building-php-cli-environment-by-docker-en
lang: en
image: /assets/images/platform/docker/building-php-cli-environment-by-docker-en/image1.png
---

In this guide, we’ll set up an environment where you can run PHP from the command line using Docker.

![Thumbnail](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Thumbnail")

## Environment
- Ubuntu 22.04.3 LTS (Running on WSL)
- Docker Engine 26.0.0

## Prerequisites
- Docker must already be installed.  
For instructions on installing Docker Engine on Windows, see [this guide](/platform/docker/installing-docker-engine-on-windows-en).

## Setup Steps
1. [Create a PHP File](#1-create-a-php-file)
2. [Start the PHP Container](#2-start-the-php-container)
3. [Execute the PHP File](#3-execute-the-php-file)

## 1. Create a PHP File
First, create a PHP file to run.  
Run the following command to create a `sample.php` file that outputs "Hello World":

```
$ echo '<?php echo "Hello World" . PHP_EOL;' > sample.php
```

The file content should look like this:

```php
<?php echo "Hello World" . PHP_EOL;
```

## 2. Start the PHP Container
Move into the directory where the `sample.php` file is located.  
Then, run the following command:

```
$ docker run -it --rm --name php-cli -w /app -v `pwd`:/app php:8.2-cli bash
```

This command starts a container named `php-cli` using the `php:8.2-cli` image and runs `bash` so you can interact with the container. The container name is arbitrary.  
Refer to a article about [docker run](/platform/docker/about-docker-commands-en#docker-run).

## 3. Execute the PHP File
Inside the PHP container, run the following.  
The `sample.php` file should be in the `/app` directory, since the directory containing it was mounted to the `/app` directory in the container.

```
/app# php sample.php
Hello World
```

The script executes successfully and outputs "Hello World".

Exit the container and the container is automatically deleted.  
Because we used the `--rm` option in the `docker run` command:

```
/app# exit
exit
$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
# (No output — the container was removed)
```

If you want to run the script again later, just repeat [Step 2](#2-start-the-php-container).

#### Run the PHP Script When Starting the Container
If you want to run the PHP script immediately when starting the container, replace `bash` in the `docker run` command from [Step 2](#2-start-the-php-container) with `php sample.php`:

```
$ docker run -it --rm --name php-cli -w /app -v `pwd`:/app php:8.2-cli php sample.php
Hello World
```

Instead of launching into an interactive shell, this runs the PHP script directly at container startup.

#### Run PHP Without Creating a File
If you don’t want to create a separate PHP file, you can skip [Step 1](#1-create-a-php-file) and run this instead:

```
$ docker run -it --rm --name php-cli -w /app -v `pwd`:/app php:8.2-cli php -r "echo 'Hello World' . PHP_EOL;"
Hello World
```

This uses the `-r` option of the `php` command to execute PHP code directly from the command line.
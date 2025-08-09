---
layout: my-post
title: "Frequently Used Docker Commands"
date: 2025-07-06 00:00:00 +0000
categories: platform docker
lang: en
---

This is a summary of Docker commands I frequently use.  
Only the commands and options I personally use are listed here.

## Environment
- Ubuntu 22.04.3 LTS (Running on WSL)
- Docker Engine 26.0.0

## List of Docker Commands
- [docker ps](#docker-ps)
- [docker rm](#docker-rm)
- [docker images](#docker-images)
- [docker rmi](#docker-rmi)
- [docker run](#docker-run)
- [docker stop](#docker-stop)
- [docker compose up](#docker-compose-up)
- [docker compose exec](#docker-compose-exec)
- [docker compose down](#docker-compose-down)

## docker ps
The `docker ps` command lists the Docker containers you have created.

Usage:  
```
docker ps [OPTIONS]
```

Example:  
```
docker ps -a
```

Options:  

|Option|Description|
|----|----|
|a|Shows all Docker containers. Without this option, only running containers are displayed.|

## docker rm
The `docker rm` command deletes the specified Docker containers.

Usage:    
```
docker rm [OPTIONS] CONTAINER [CONTAINER...]
```

Example:  
```
docker rm <containerID1> <containerID2>
```

Arguments:

|Argument|Description|
|----|----|
|CONTAINER|Specify the container ID. Multiple IDs can be specified. You can check container IDs with `docker ps`.|

Options:

|Option|Description|
|----|----|
|f|Forcefully deletes Docker containers. This can also remove running containers.|

## docker images
The `docker images` command lists Docker images that have been built or downloaded.

Usage:   
```
docker images [OPTIONS] [REPOSITORY[:TAG]]
```

Example:  
```
docker images
```

## docker rmi
The `docker rmi` command removes the specified Docker images.

Usage:  
```
docker rmi [OPTIONS] IMAGE [IMAGE...]
```

Example 1:  
```
docker rmi <imageID1> <imageID2>
```

Example 2:  
```
docker rmi php:8.2
```

Arguments:  

|Argument|Description|
|----|----|
|IMAGE|Specify the image ID. Multiple IDs can be specified. You can check image IDs with `docker images`. If there are multiple tags for the same image ID, you can delete a specific tag by using `<repository>:<tag>` instead. Example 2 shows this with `php:8.2`.|

## docker run
The `docker run` command launches a Docker container from a Docker image.

Usage:  
```
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

Example:  
```
docker run -it --rm --name php-cli -w /app -v `pwd`:/app php:8.2-cli bash
```

Arguments:

|Argument|Description|
|----|----|
|IMAGE|Specify the Docker image. In the example, `php:8.2-cli` is used.|
|COMMAND|The command to run after the container starts. In the example, `bash` is used to connect to the container.|

Options: 

|Option|Description|
|----|----|
|i|Keeps STDIN open, allowing input to the Docker container.|
|t|Allocates a pseudo-TTY. TTY is a device that enables input/output within the container. Keeping TTY active prevents the container from stopping while connected.|
|rm|Automatically removes the container when it exits.|
|name|Assigns a name to the container. In the example, `php-cli` is specified.|
|w|Sets the working directory inside the container. In the example, `/app` is used.|
|v|Mounts a host directory to the container. The example mounts the current host directory to `/app` inside the container using the `pwd` command.|
|p|Forward a host's port to the Docker container's port. For example, `-p 8080:80` (`8080` is the host's port, `80` is the container's port)|

## docker stop
The `docker stop` command stops running Docker containers.

Usage:  
```
docker stop [OPTIONS] CONTAINER [CONTAINER...]
```

Example:  
```
docker stop <containerID>
```

Arguments:  

|Argument|Description|
|----|----|
|CONTAINER|Specify the container ID. Multiple IDs can be specified. You can check container IDs with `docker ps`.|

## docker compose up
The `docker compose up` command starts multiple containers as defined in the `docker-compose.yml` file.

Usage:  
```
docker compose up [OPTIONS] [SERVICE...]
```

Example 1:  
```
docker compose up -d
```

Example 2:  
```
docker compose up --build
```

Options:  

|Option|Description|
|----|----|
|d|Starts the containers in the background.|
|build|Builds the Docker images before starting the containers.|

## docker compose exec
The `docker compose exec` command runs a command inside a running container defined in `docker-compose.yml`. A pseudo-TTY is allocated by default.

Usage:  
```
docker compose exec [OPTIONS] SERVICE COMMAND [ARGS...]
```

Example:  
```
docker compose exec <service-name> bash
```

Arguments:  

|Argument|Description|
|----|----|
|COMMAND|The command to run inside the container. In the example, `bash` is used to connect to the container.|

## docker compose down
The `docker compose down` command stops and removes all containers defined in the docker-compose.yml file.

Usage:  
```
docker compose down [OPTIONS] [SERVICES]
```

Example 1:  
```
docker compose down
```

Example 2:  
```
docker compose down -v
```

Options:  

|Option|Description|
|----|----|
|v|Removes the Docker volumes defined in `docker-compose.yml`.|
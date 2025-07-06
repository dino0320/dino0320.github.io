---
layout: my-post
title: "About Docker depends on"
date: 2025-07-06 00:00:00 +0000
categories: platform docker
page_name: about-docker-depends-on-en
lang: en
---

I investigated how Docker’s depends_on works.

## Environment
- Docker Engine 26.0.0
- Docker Compose version v2.25.0

## What is depends_on?
In Docker, the `depends_on` option defines dependencies between services for startup and shutdown order.  
You can define `depends_on` in a `docker-compose.yml` file.

For example, if a web server container needs to interact with a database container during startup, the database container must start before the web server.  
By using `depends_on`, you can specify this startup order.

There are two ways to write `depends_on`: short syntax and long syntax.

### Short syntax
The short syntax simply lists the names of dependent services.

Example (quoted from the [official documentation](https://docs.docker.com/compose/compose-file/05-services/#depends_on)):

```yml
services:
  web:
    build: .
    depends_on: # Define depends on
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```

In this example:  
The `web` service will start after the `db` and `redis` services and stop before they stop.

However, with short syntax, Docker does not wait for the dependent services to be fully ready (e.g., a database accepting connections).  
So even though the `db` container starts before the `web` container, if the database inside `db` isn't ready yet, the `web` service might fail when trying to connect to it.

### Long syntax
The long syntax provides more control and lets you configure the following additional options:

|Field|Description|
|----|----|
|restart|Whether to restart this service if the dependent service is restarted (e.g., via `docker compose restart`). Default is `false`.|
|condition|Specifies the state the dependency must reach before this service starts. More on this below.|
|required|If `false`, the service will start even if the dependency fails to start. Default is `true`.|

You can specify one of the following dependency states with `condition`:

|State|Description|
|----|----|
|service_started|Starts this service once the dependency has started. This behaves like the [short syntax](#short-syntax).|
|service_healthy|Starts this service after the dependency has passed its healthcheck. See [About Docker healthcheck](/platform/docker/about-docker-healthcheck-en) for more details.|
|service_completed_successfully|Starts this service only after the dependency has exited successfully.|

Here is an example using long syntax (also from the [official documentation](https://docs.docker.com/compose/compose-file/05-services/#depends_on)):

```yml
services:
  web:
    build: .
    depends_on: # Define depends on
      db:
        condition: service_healthy
        restart: true
      redis:
        condition: service_started
  redis:
    image: redis
  db:
    image: postgres
```

Explanation:  
The `web` service will start after the `db` service passes its healthcheck.  
The `web` service will restart if the `db` service is restarted.  
The `web` service will start after the `redis` service has started.  
The `web` service will stop before the `db` and `redis` services stop.
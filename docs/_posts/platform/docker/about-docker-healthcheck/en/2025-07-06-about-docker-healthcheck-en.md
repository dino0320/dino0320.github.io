---
layout: my-post
title: "About Docker Healthcheck"
date: 2025-07-06 00:00:00 +0000
categories: platform docker
page_name: about-docker-healthcheck-en
lang: en
image: /assets/images/platform/docker/about-docker-healthcheck-en/image1.png
---

I looked into how a Docker healthcheck works.

![Thumbnail](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Thumbnail")

## Environment
- Docker Engine 26.0.0
- Docker Compose version v2.25.0

## What is a Healthcheck?
A Docker healthcheck is a way to determine whether a container is healthy (i.e., operating normally).
Specifically, a command is executed at regular intervals, and if it succeeds, the container is considered healthy. If it fails a certain number of times in a row, the container is marked as unhealthy. (The container's initial state is starting.)

Healthchecks can be defined in either a `Dockerfile` or a `docker-compose.yml` file.
If a healthcheck is defined in `docker-compose.yml`, it will override any definition from the `Dockerfile`.

The following are the available healthcheck fields in `docker-compose.yml`:

|Field|Description|
|----|----|
|test|The command executed to check the container's health. A return status of 0 indicates success, while 1 indicates failure.|
|interval|The interval between each healthcheck execution.|
|timeout|The timeout duration for a single healthcheck. If the command takes longer than this, it's considered a failure.|
|retries|The number of consecutive failures allowed before the container is marked as `unhealthy`.|
|start_period|Initialization period during container startup. During this period, failures do not count toward the `unhealthy` threshold. If a healthcheck succeeds during this period, the initialization ends immediately.|
|start_interval|The interval at which healthchecks are performed during the `start_period`.|

Here is an example of a healthcheck definition:

```yml
healthcheck:
  test: curl -f https://localhost || exit 1
  interval: 1m30s
  timeout: 10s
  retries: 3
  start_period: 40s
  start_interval: 5s
```

Explanation:    
`curl -f https://localhost || exit 1` is the healthcheck command.  
During the 40-second `start_period`, the healthcheck is run every 5 seconds.   
After initialization ends, the healthcheck runs every 1 minute and 30 seconds.  
Each check times out after 10 seconds.  
If the check fails 3 times in a row, the container is considered `unhealthy`.
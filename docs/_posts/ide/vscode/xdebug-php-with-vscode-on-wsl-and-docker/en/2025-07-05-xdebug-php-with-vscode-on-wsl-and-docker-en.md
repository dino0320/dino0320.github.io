---
layout: my-post
title: "Debugging PHP Code Running on WSL + Docker Using Xdebug in VSCode"
date: 2025-07-05 00:00:00 +0000
categories: ide vscode
page_name: xdebug-php-with-vscode-on-wsl-and-docker-en
lang: en
image: /assets/images/ide/vscode/xdebug-php-with-vscode-on-wsl-and-docker-en/image9.png
---

This guide explains how to set up Visual Studio Code (VSCode) to debug PHP code running inside a Docker container on WSL using Xdebug.

![Thumbnail](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image9.png "Thumbnail")

## Environment
- Windows 10 64-bit
- Ubuntu 22.04.3 LTS (running on WSL)
- Docker Engine 26.0.0
- PHP 8.2.15
- Visual Studio Code 1.87.2

## Prerequisites
- You have a PHP project running in a Docker container on WSL (using Docker Compose).  
For instructions on setting up a Laravel project in this environment, see [this guide](/web-application-framework/laravel/running-laravel-project-on-nginx-en).
- Visual Studio Code is installed.

## Setup Steps
1. [Connect to WSL in VSCode](#1-connect-to-wsl-in-vscode)
2. [Install Xdebug](#2-install-xdebug)
3. [Install the PHP Debug Extension in VSCode](#3-install-the-php-debug-extension-in-vscode)
4. [Verify Xdebug Works](#4-verify-xdebug-works)

## 1. Connect to WSL in VSCode
For instructions on how to connect VSCode to WSL, see [this article](/ide/vscode/connecting-to-wsl-with-vscode-en).

## 2. Install Xdebug
Install Xdebug in your Docker container.

Update `docker-compose.yml` to add the host definition.  
Note: If you’re using Docker Desktop, this may not be necessary.

`docker-compose.yml` :
```yml
services:
  web:
    build: ./docker/web
    volumes:
      - .:/srv/example.com
    ports:
      - 8080:80
    extra_hosts:
      - host.docker.internal:host-gateway    # Add this line
```

Add Xdebug Configuration to PHP settings.  
Create the following file. 

`99-xdebug.ini` : 
```ini
[xdebug]
zend_extension=/usr/lib64/php8.2/modules/xdebug.so    # Adjust path to actual .so file
xdebug.mode = debug    # Enables a remote debug
xdebug.start_with_request = yes    # Enables a remote debug
xdebug.client_host = host.docker.internal    # Set the host Xdebug connects
xdebug.client_port = 9003    # Set a port Xdebug connects
xdebug.log = /tmp/xdebug.log    # Set a log path
```

Note:  
`zend_extension`: If you're unsure of the path, install Xdebug manually in the container and check the correct path.  
`xdebug.client_host`: The host Xdebug in a Docker container connects is the host running Docker, so specify host.docker.internal.   
`xdebug.client_port`: Default port is 9003, but you can change it if needed.  
`xdebug.log`: Set to any writable path in the container.

Add Xdebug installation to your `Dockerfile`.
I used `amazonlinux` as a Docker image. 

`Dockerfile` :
```dockerfile
FROM amazonlinux:2023

...

# Install Xdebug
RUN yum -y install php-pear php-devel
RUN pecl install xdebug-3.3.1
COPY php/conf.d/99-xdebug.ini /etc/php.d/99-xdebug.ini

...
```

Ensure the path in COPY matches where your `99-xdebug.ini` is located relative to the Dockerfile.

Verify Xdebug Installation.  
Connect to the container and run the following commands.

```
# php -i | grep xdebug
```

If you see Xdebug configuration listed, the installation was successful.

## 3. Install the PHP Debug Extension in VSCode
Install the PHP Debug extension in VSCode.

Open VSCode and click on Extensions.

![Extensions Button](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png)

Search for `php debug` and install the one named "PHP Debug".

![PHP Debug Extension](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image2.png)

Create `launch.json`.  
Click on "Run and Debug".

![Run and Debug Button](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image3.png)

Click "Create a launch.json file".

![Create launch.json Link](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image4.png)

Select "PHP" as the debugger.

![Choose Debugger](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image5.png)

Add `pathMappings` to the generated `launch.json`.  
This maps the directory inside the Docker container to your local workspace folder.

```json
{
    ...

    "version": "0.2.0",
    "configurations": [
        {
            "name": "Listen for Xdebug",
            "type": "php",
            "request": "launch",
            "port": 9003,
            // Add pathMappings
            "pathMappings": {
                "/srv/example.com": "${workspaceFolder}"
            }
        },

    ...
    ]
}
```

Now the PHP Debug extension is configured and ready to use.

## 4. Verify Xdebug Works
Start Docker Container.

In VSCode, run the "Listen for Xdebug" configuration you created in `launch.json`.

![Start Debug Button](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image6.png)

Open a PHP file that can be accessed via the browser and set a breakpoint.

![Breakpoint](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image7.png)

Access the PHP file from your browser.

![Execution Stopped at Breakpoint](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image8.png)

If the execution stops at the breakpoint, Xdebug is working correctly.
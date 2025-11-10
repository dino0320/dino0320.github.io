---
layout: my-post
title: "Installing Composer During Docker Image Build"
date: 2025-07-08 00:00:00 +0000
categories: programming php
page_name: installing-composer-on-docker-en
lang: en
---

We'll configure our Docker image to install Composer during the build process.

## Environment
- Windows 10 64-bit
- Ubuntu 22.04.3 LTS (running on WSL)
- Docker Engine 26.0.0
- Amazon Linux 2023(used as the OS for the Docker container)

## Installation Steps
1. [Create a Script to Install Composer](#1-create-a-script-to-install-composer)
2. [Install Composer During the Docker Image Build](#2-install-composer-during-the-docker-image-build)
3. [Verify That Composer Is Installed](#3-verify-that-composer-is-installed)

## 1. Create a Script to Install Composer
We'll start by creating a script to install Composer.

Refer to [this page](https://getcomposer.org/doc/faqs/how-to-install-composer-programmatically.md) to create the script.  
Save it as `install-composer.sh`.

`install-composer.sh` :

```bash
#!/bin/bash

set -euxo pipefail

# Get the expected checksum
EXPECTED_CHECKSUM="$(php -r 'copy("https://composer.github.io/installer.sig", "php://stdout");')"
# Download the Composer installer
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
# Generate the actual checksum
ACTUAL_CHECKSUM="$(php -r "echo hash_file('sha384', 'composer-setup.php');")"

# Exit with error if checksums don't match
if [ "$EXPECTED_CHECKSUM" != "$ACTUAL_CHECKSUM" ]
then
    >&2 echo 'ERROR: Invalid installer checksum'
    rm composer-setup.php
    exit 1
fi

# Allow optional version specification via the second argument
VERSION_OPTION=''
if [ $# == 2 ]
then
    VERSION_OPTION="--version=$2"
fi

# Install Composer with optional version
php composer-setup.php $VERSION_OPTION
rm composer-setup.php

# Move Composer to the path specified by the first argument
COMPOSER_PATH=$1
mv composer.phar $COMPOSER_PATH
```

## 2. Install Composer During the Docker Image Build
We'll configure the Docker build to install Composer.

Use the script from [Step 1](#1-create-a-script-to-install-composer) (`install-composer.sh`) and run it during the image build.  
Place the `Dockerfile` in the same directory as the script.

`Dockerfile` :
```dockerfile
# Use Amazon Linux 2023
FROM amazonlinux:2023

# Install PHP and unzip (required for Composer)
RUN yum -y install php8.2 unzip-6.0

# Copy the install-composer.sh script to a temporary location
COPY install-composer.sh /tmp/install-composer.sh
# Run the script with the destination path and version
RUN /tmp/install-composer.sh /usr/local/bin/composer 2.7.6
```

## 3. Verify That Composer Is Installed
Check whether Composer was successfully installed.

Run the following commands in the same directory as the `Dockerfile`:

```bash
$ docker build . -t installed_composer
$ docker run -it --rm --name composer_verification installed_composer bash
```

These commands will build the Docker image named `installed_composer` and launch a container named `composer_verification`.

Inside the container, run:

```
# composer -v
   ______
  / ____/___  ____ ___  ____  ____  ________  _____
 / /   / __ \/ __ `__ \/ __ \/ __ \/ ___/ _ \/ ___/
/ /___/ /_/ / / / / / / /_/ / /_/ (__  )  __/ /
\____/\____/_/ /_/ /_/ .___/\____/____/\___/_/
                    /_/
Composer version 2.7.6 2024-05-04 23:03:15
```

If you see the Composer version output, it confirms that Composer has been successfully installed.
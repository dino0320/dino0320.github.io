---
layout: my-post
title: "Installing Node.js During Docker Image Build"
date: 2025-07-06 00:00:00 +0000
categories: programming javascript
page_name: installing-node-on-docker-en
lang: en
---

This guide explains how to install Node.js during the Docker image build process.

## Reference
- [Download Node.js® - Node.js](https://nodejs.org/en/download/package-manager)

## Environment
- Windows 10 64-bit
- Ubuntu 22.04.3 LTS (running on WSL)
- Docker Engine 26.0.0
- Amazon Linux 2023(Docker container OS)

## Installation Steps
1. [Create a Script to Install Node.js](#1-create-a-script-to-install-nodejs)
2. [Install Node.js During Docker Image Build](#2-install-nodejs-during-docker-image-build)
3. [Verify the Installation](#3-verify-the-installation)

## 1. Create a Script to Install Node.js
Create a script that installs Node.js using nvm (Node Version Manager).  
Refer to [this page](https://nodejs.org/en/download/package-manager) for guidance.  
Save the script as `install-node.sh`.

`install-node.sh` :
```bash
#!/bin/bash

set -euxo pipefail

# Download and run the nvm installation script
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# Load nvm
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

# Install Node.js with the specified version (as first argument)
VERSION=$1
# Install Node.js
nvm install $VERSION
```

## 2. Install Node.js During Docker Image Build
Modify the Docker build process to install Node.js using the [script above](#1-create-a-script-to-install-nodejs) (`install-node.sh`).

Place the following `Dockerfile` in the same directory as `install-node.sh`.

`Dockerfile` :
```dockerfile
# Use Amazon Linux 2023
FROM amazonlinux:2023

# Install tar and gzip, required by nvm
RUN yum -y install tar-1.34 gzip-1.12

# Create .bashrc file
# This allows nvm to auto-load in bash sessions
RUN touch /root/.bashrc
# Copy the install script into the location of your choice
COPY install-node.sh /tmp/install-node.sh
# Run the script and install Node.js (specify version)
RUN /tmp/install-node.sh 20.14.0
```

## 3. Verify the Installation
Run the following commands in the same directory as the `Dockerfile`:

```bash
$ docker build . -t installed_node
$ docker run -it --rm --name node_verification installed_node bash
```

This builds a Docker image named `installed_node` and starts a container named `node_verification`.

Inside the container, run:

```
# node -v
v20.14.0
# npm -v
10.7.0
```

You should see the installed versions of Node.js and npm, confirming that the installation was successful.
---
layout: my-post
title: "Deploy a Laravel project to ECS using Fargate"
date: 2025-07-31 00:00:00 +0000
categories: aws ecs
page_name: deploy-laravel-to-ecs-using-fargate-en
lang: en
---

This article explains how to deploy a Laravel project to Amazon ECS on AWS using Fargate.

## Reference
- [Pushing a Docker image to an Amazon ECR private repository](https://docs.aws.amazon.com/AmazonECR/latest/userguide/docker-push-ecr-image.html)
- [Learn how to create an Amazon ECS Linux task for the Fargate launch type](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/getting-started-fargate.html)
- [Amazon ECS task execution IAM role](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_execution_IAM_role.html)
- [Amazon ECR interface VPC endpoints (AWS PrivateLink)](https://docs.aws.amazon.com/AmazonECR/latest/userguide/vpc-endpoints.html)

## Prerequisites
- An AWS account
- A user in IAM Identity Center  
For more information, follow [this instruction](/aws/iam/create-user-with-administrative-access-in-iam-identity-center-en).
- AWS CLI already installed  
For more information, follow [this instruction](/aws/cli/install-aws-cli-on-linux-en)
- An SSO session and a profile with AWS CLI  
For more information, follow [this instruction](/aws/cli/configure-sso-session-and-profile-with-aws-cli-en)

## Environment
- Ubuntu 22.04.3 LTS (running on WSL)
- Docker Engine 26.0.0
- Laravel 11

## Architecture Diagram
This is an architecture diagram for the system we'll build.

![Architecture Diagram](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image37.png "Architecture Diagram")

## Deployment Steps
1. [Create Laravel Project](#1-create-laravel-project)
2. [Create Repository](#2-create-repository)
3. [Push Docker Image](#3-push-docker-image)
4. [Create Cluster](#4-create-cluster)
5. [Create ECS Task Execution IAM Role](#5-create-ecs-task-execution-iam-role)
6. [Create Task Definition](#6-create-task-definition)
7. [Create Security Group](#7-create-security-group)
8. [Create VPC Endpoints](#8-create-vpc-endpoints)
9. [Create Application Load Balancer](#9-create-application-load-balancer)
10. [Create Service](#10-create-service)
11. [Verify Laravel Project](#11-verify-laravel-project)

## 1. Create Laravel Project
Create a Laravel project.  
For more information, refer to [this article](/web-application-framework/laravel/running-laravel-project-on-nginx-en).

If you use the above example, modify the `Dockerfile` as follows:  
(Add a process to copy the project sources into the container and adjust paths when copying files)


`Dockerfile`:

```Dockerfile
FROM amazonlinux:2023

# (Added) Copy this project source to the container
COPY . /srv/example.com

# (Changed) Adjust paths when copying the files
# e.g., nginx/nginx.repo /etc/yum.repos.d/nginx.repo -> docker/web/nginx/nginx.repo /etc/yum.repos.d/nginx.repo

# Install NGINX
RUN yum -y install yum-utils
COPY docker/web/nginx/nginx.repo /etc/yum.repos.d/nginx.repo
RUN yum -y install nginx-1.24.0
COPY docker/web/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf

# Install php-fpm
RUN yum -y install php8.2-fpm php8.2-mbstring
# Not creating the directory in advance will cause an error
RUN mkdir /run/php-fpm
RUN mkdir /var/run/php
COPY docker/web/php/php-fpm.d/zzz-www.conf /etc/php-fpm.d/zzz-www.conf
```

Also modify the `docker-compose.yml` file if you plan to run the container locally.  
Change the path where the container is built and specify the `Dockerfile` path:  
(The container needs to be built in the project root path to copy the sources)

`docker-compose.yml`:

```yml
services:
  web:
    # Change the "build" field
    build:
      context: .
      dockerfile: ./docker/web/Dockerfile
    volumes:
      - .:/srv/example.com
    ports:
      - "8080:80"
    command: bash -c "chmod 755 /srv/example.com/docker/web/start.sh && /srv/example.com/docker/web/start.sh"
```

## 2. Create Repository
Navigate to [ECR on AWS Management Console](https://console.aws.amazon.com/ecr/).  
Click the "Create" button.

![ECR on AWS Management Console](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "ECR on AWS Management Console")

Enter a repository name of your choice and click the "Create" button.  
In this example, specify "laravel-project-repository".

![Repository creation](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image2.png "Repository creation")

## 3. Push Docker Image
Build the Docker image with the registry format:

```bash
$ docker build -t <Your AWS account ID>.dkr.ecr.<Your ECR repository region>.amazonaws.com/laravel-project-repository .
```

Specify the `-f` option if your `Dockerfile` is not in the project root directory:

```bash
$ docker build -t <Your AWS account ID>.dkr.ecr.<Your ECR repository region>.amazonaws.com/laravel-project-repository -f <Your Dockerfile path> .
```

If you already have a Docker image, retag it as follows:

```bash
$ docker tag <Your Docker image repository or image ID>:<Your Docker image tag> <Your AWS account ID>.dkr.ecr.<Your ECR repository region>.amazonaws.com/laravel-project-repository:latest
```

Authenticate your Docker client with the Amazon AWS ECS registry and run `docker login`:

```bash
$ aws ecr get-login-password --region <Your ECR repository region> | docker login --username AWS --password-stdin <Your AWS account ID>.dkr.ecr.<Your ECR repository region>.amazonaws.com
```

Push your Docker image to your ECR repository:

```bash
$ docker push <Your AWS account ID>.dkr.ecr.<Your ECR repository region>.amazonaws.com/laravel-project-repository:latest
```

You can check your image on Amazon Management Console.

![Your images](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image3.png "Your images")

## 4. Create Cluster
Navigate to [ECS on AWS Management Console](https://console.aws.amazon.com/ecs/).  
Click the "Create Cluster" button.

![Clusters on AWS Management Console](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image4.png "Clusters on AWS Management Console")

Enter any cluster name and check the "AWS Fargate (serverless)" checkbox.  
In this example, specify "laravel-project-cluster".  
Then click the "Create" button.

![Cluster creation](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image5.png "Cluster creation")

## 5. Create ECS Task Execution IAM Role
To grant containers and Fargate agents permission to use AWS services, create an ECS task execution IAM role.

Navigate to IAM on [IAM on AWS Management Console](https://console.aws.amazon.com/iam/) and click "Roles".

![IAM on AWS Management Console](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image8.png "IAM on AWS Management Console")

Click the "Create role" button.

![Roles on AWS Management Console](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image9.png "Roles on AWS Management Console")

Select "AWS Service", then choose "Elastic Container Service Task".  
Click the "Next" button.

![Select trusted entity screen1](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image10.png "Select trusted entity screen1")

![Select trusted entity screen2](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image11.png "Select trusted entity screen2")

Select the "AmazonECSTaskExecutionRolePolicy" policy.  
You can add additional policies as needed (e.g., S3 access if required during container builds).  
Click the "Next" button.

![Add permissions screen](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image12.png "Add permissions screen")

Enter "ecsTaskExecutionRole" in the Role name field (you can choose any name).  
Click the "Create role" button.

![Final screen](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image13.png "Final screen")

## 6. Create Task Definition
A task definition is used to create ECS tasks.  
In this example, we'll create a task definition for [this project](#1-create-laravel-project).

Navigate to "Task definitions".

![ECS on AWS Management Console](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image6.png "ECS on AWS Management Console")

Click the "Create new task definition with JSON" option.

![Task definitions](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image7.png "Task definitions")

Enter any value into "family"(`laravel-project-fargate`) and "containerDefinitions.name"(`web`) fields.  
Retrieve your image URI and enter it in the "containerDefinitions.image" field.  
Add the "portMapping" and "command" items.

`task definition`:

```json
{
    "requiresCompatibilities": [
        "FARGATE"
    ],
    "family": "laravel-project-fargate",
    "containerDefinitions": [
        {
            "name": "web",
            "image": "<Your AWS account ID>.dkr.ecr.<Your ECS repository region>.amazonaws.com/laravel-project-repository:latest",
            "portMappings": [
                {
                    "containerPort": 80,
                    "protocol": "tcp"
                }
            ],
            "essential": true,
            "command": [
                "/bin/sh",
                "-c",
                "chmod 755 /srv/example.com/docker/web/start.sh && /srv/example.com/docker/web/start.sh"
            ]
        }
    ],
    "volumes": [],
    "networkMode": "awsvpc",
    "memory": "3 GB",
    "cpu": "1 vCPU",
    "executionRoleArn": "ecsTaskExecutionRole"
}
```

Click the "Create" button.

## 7. Create Security Group
Go to [VPC on AWS Management Console](https://console.aws.amazon.com/vpc/), then click "Security groups".

![VPC on AWS Management Console](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image36.png "VPC on AWS Management Console")

Click "Create security group".  
Create the following security groups.

### For private

Inbound rules:

| Type | Port range | Source | Note |
|------|------------|--------|------|
| All traffic | All | This security group ID | To access VPC endpoints. |
| All traffic | All | Security group ID for public | To allow the Load Balancer to access ECS tasks. |

Outbound rules:

| Type | Port range | Source | Note |
|------|------------|--------|------|
| HTTPS | 443 | 0.0.0.0/0 | To pull ECR images. |

### For public

Inbound rules:

| Type | Port range | Source | Note |
|------|------------|--------|------|
| HTTP | 80 | 0.0.0.0/0 | To allow access to the Load Balancer. |

Outbound rules:

| Type | Port range | Source | Note |
|------|------------|--------|------|
| All traffic | All | Security group ID for private | To allow access to ECS tasks. |

## 8. Create VPC Endpoints
To pull ECR images from ECS tasks in private subnets.  
In this example, the following three endpoints are required.

- `com.amazonaws.region.ecr.dkr`
- `com.amazonaws.region.ecr.api`
- `com.amazonaws.region.s3` (Gateway)  
This is because ECR uses Amazon S3 to store image layers.

Go to [VPC on AWS Management Console](https://console.aws.amazon.com/vpc/).  
Click "Endpoints".

![VPC on AWS Management Console](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image32.png "VPC on AWS Management Console")

Click "Create endpoint".

![Endpoints](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image33.png "Endpoints")

Enter a name and select "AWS services" for the Type.

![Endpoint creation1](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image34.png "Endpoint creation1")

Choose `com.amazonaws.region.ecr.dkr`.

![Endpoint creation2](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image35.png "Endpoint creation2")

Select your VPC, private subnets and security group for private.

Click "Create endpoint".

Repeat the same steps for the others.

## 9. Create Application Load Balancer
Create an Application Load Balancer to distribute incoming traffic across Fargate containers.

Go to [EC2 on AWS Management Console](https://console.aws.amazon.com/ec2/).  
Click "Load Balancers".

![EC2 on AWS Management Console](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image21.png "EC2 on AWS Management Console")

Click "Create Application Load Balancer".

![Load balancers on AWS Management Console](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image22.png "Load balancers on AWS Management Console")

Enter a Load Balancer name, choose "Internet-facing" for the Scheme, and select "IPv4" for the IP address type.

![Load balancer creation1](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image23.png "Load balancer creation1")

Select your VPC, public subnets and security group for public.

Then click "Create target group".

![Load balancer creation2](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image24.png "Load balancer creation2")

Choose "IP addresses" as the target type.

![Target group creation1](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image25.png "Target group creation1")

Enter "Target group name" and specify "HTTP" for the Protocol and "80" for the Port.  
Select "IPv4" for the IP address type.

![Target group creation2](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image26.png "Target group creation2")

Select your VPC.

Configure the Health checks as follows.

![Target group creation3](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image27.png "Target group creation3")

Click "Next", then click "Create target group".

Return to the Load Balancer creation screen.  
Select the target group you just created as the listener’s target.

![Load balancer creation3](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image28.png "Load balancer creation3")

Click "Create load balancer".

## 10. Create Service
Return to your [clusters screen on AWS Management Console](https://console.aws.amazon.com/ecs/clusters/).  
Click your cluster.

![Clusters on AWS Management Console](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image14.png "Clusters on AWS Management Console")

Click the "Create" button.

![Services on AWS Management Console](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image15.png "Services on AWS Management Console")

Enter the task definition family; the latest revision and service name will be populated automatically.  
In this example, the service name was changed as follows.

![Service Creation](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image16.png "Service Creation")

Select your VPC, private subnets and security group for private.

Then turn the "Public IP" button off.

![Network on Service Creation](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image17.png "Network on Service Creation")

Then configure the Load balancing.  
Specify the load balancer you created.

![Load balancing on Service Creation1](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image30.png "Load balancing on Service Creation1")

![Load balancing on Service Creation2](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image31.png "Load balancing on Service Creation2")

Click the "Create" button.

## 11. Verify Laravel Project
Retrieve the DNS name from the "Details" section of your load balancer.

Access `http://<Your load balancer DNS name>` in your browser (e.g. `http://laravel-project-load-balancer-xxxxx.xxxxx.elb.amazonaws.com`).  
If you see the following web page, your Laravel project was successfully deployed to ECS.

![Laravel project web page](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image20.png "Laravel project web page")
---
layout: my-post
title: "How to Access External Services from a Private ECS Fargate Container (NAT Gateway)"
date: 2026-01-17 00:00:00 +0000
categories: aws ecs
page_name: how-to-access-external-services-from-private-ecs-container-en
lang: en
image: /assets/images/aws/ecs/how-to-access-external-services-from-private-ecs-container-en/image1.png
---

This article explains how to access external services from a private ECS Fargate container via a NAT Gateway.

![Thumbnail](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Thumbnail")

## Prerequisite
- Your project is already deployed to ECS  
For more information, follow [this instruction](/aws/ecs/deploy-laravel-to-ecs-using-fargate-en).

## Workflow
1. [Create NAT Gateway](#1-create-nat-gateway)
2. [Edit Private Route Table](#2-edit-private-route-table)

## 1. Create NAT Gateway
Go to [NAT Gateways on AWS Management Console](https://console.aws.amazon.com/vpcconsole/home#NatGateways).  
Click "Create NAT Gateway".

![NAT Gateways on AWS Management Console](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image2.png "NAT Gateways on AWS Management Console")

Enter a name and select your VPC.  
"Regional" is set to "Availability mode" by default. If you need to set it up for a specific availability zone, select "Zonal" and configure it accordingly.  
Also, "Automatic" is already set to "Method of Elastic IP (EIP) allocation". Select "Manual" if you want to use Elastic IP addresses that you prepared in advance.  
Make sure "Connectivity type" is "Public".  
Then Click "Create NAT Gateway".

![NAT Gateway creation](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image3.png "NAT Gateway creation")

## 2. Edit Private Route Table
Go to [Route tables on AWS Management Console](https://console.aws.amazon.com/vpcconsole/home#RouteTables).  
Select your private route table and click "Edit routes".

![Private route table](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image4.png "Private route table")

Add the following route and click "Save changes".  
This configuration sends all outbound traffic to the internet via the NAT Gateway.

|Destination|Target|
|-----------|------|
|0.0.0.0/0|NAT Gateway ([created above](#1-create-nat-gateway))|

![Route table edit screen](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image5.png "Route table edit screen")

With this setup, private ECS containers can access external services such as external APIs, Amazon S3, and other AWS services.
Note: For AWS services such as Amazon S3, using a VPC Endpoint is recommended instead of a NAT Gateway.
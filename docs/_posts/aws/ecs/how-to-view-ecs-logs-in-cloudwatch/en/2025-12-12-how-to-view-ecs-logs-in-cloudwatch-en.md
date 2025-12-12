---
layout: my-post
title: "How to View ECS Logs in CloudWatch"
date: 2025-12-12 00:00:00 +0000
categories: aws ecs
page_name: how-to-view-ecs-logs-in-cloudwatch-en
lang: en
image: /assets/images/aws/ecs/how-to-view-ecs-logs-in-cloudwatch-en/image1.png
---

This article explains how to view ECS logs in CloudWatch.

![Thumbnail](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Thumbnail")

## References
- [Example Amazon ECS task definition: Route logs to CloudWatch](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/specify-log-config.html)
- [Using CloudWatch Logs with interface VPC endpoints](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/cloudwatch-logs-and-interface-VPC.html)

## Prerequisite
- Your project is already deployed to ECS  
For more information, follow [this instruction](/aws/ecs/deploy-laravel-to-ecs-using-fargate-en).

## Configuration Steps
1. [Add logConfiguration](#1-add-logconfiguration)
2. [Create VPC Endpoint](#2-create-vpc-endpoint)
3. [Edit ECS Task Execution IAM Role](#3-edit-ecs-task-execution-iam-role)

## 1. Add logConfiguration
Add a `logConfiguration` section to your `task-definition.json` file:

`task-definition.json`:

```json
{
    "requiresCompatibilities": [
        "FARGATE"
    ],
    "family": "my-fargate",
    "containerDefinitions": [
        {
            "name": "my-container",
            ...
            // Add logConfiguration
            "logConfiguration": {
                "logDriver": "awslogs",
                "options": {
                    "awslogs-create-group": "true",
                    "awslogs-group": "my-log-group",
                    "awslogs-region": "ap-northeast-1",
                    "awslogs-stream-prefix": "my-container"
                }
            },
            ...
```

Here are the explanations for each option:

| Option | Description |
|--------|-------------|
| awslogs-create-group | When set to `true`, the log group is created automatically. |
| awslogs-group | The name of the log group where logs will be sent. |
| awslogs-region | The region where the log group exists. |
| awslogs-stream-prefix | The prefix used for log streams. |

## 2. Create VPC Endpoint
Create a VPC Endpoint for CloudWatch Logs if your ECS containers are in private subnets.

Go to [VPC on AWS Management Console](https://console.aws.amazon.com/vpc/).  
Click "Endpoints".

![VPC on AWS Management Console](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image2.png "VPC on AWS Management Console")

Click "Create endpoint".

![Endpoints](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image3.png "Endpoints")

Enter a name and select "AWS services" for the Type.

![Endpoint creation1](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image4.png "Endpoint creation1")

Choose `com.amazonaws.region.logs`.

![Endpoint creation2](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image5.png "Endpoint creation2")

Select your VPC, private subnets and security group for private.

Click "Create endpoint".

## 3. Edit ECS Task Execution IAM Role
Add a policy that allows the ECS task execution IAM role to create log groups.  
For details about this IAM role, see [this article](/aws/ecs/deploy-laravel-to-ecs-using-fargate-en#5-create-ecs-task-execution-iam-role).

Add a policy like the following:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "logs:CreateLogGroup",
            "Resource": "arn:aws:logs:<Your log group region>:<Your Account ID>:log-group:*"
        }
    ]
}
```

If your ECS task execution IAM role doesn't include `logs:CreateLogStream` and `logs:PutLogEvents`, you may need to add them to the policy above.

Once everything is set up, you should be able to view ECS logs in CloudWatch after deploying your service.
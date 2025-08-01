---
layout: my-post
title: "Automated Deployment to ECS Using GitHub Actions"
date: 2025-08-01 00:00:00 +0000
categories: aws ecs
page_name: automated-deployment-to-ecs-using-github-actions-en
lang: en
---

This article explains how to automate deployment to Amazon ECS on AWS using GitHub Actions.

## Reference
- [Deploying to Amazon Elastic Container Service](https://docs.github.com/en/actions/how-tos/deploy/deploy-to-third-party-platforms/amazon-elastic-container-service)
- [Configuring OpenID Connect in Amazon Web Services](https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-aws)
- [Configuring a role for GitHub OIDC identity provider](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-idp_oidc.html#idp_oidc_Create_GitHub)
- [GitHub Actions から AWS へアクセスする](https://qiita.com/yh1224/items/2a8223201b48a5c41e7a)

## Prerequisites
- An AWS account
- A user in IAM Identity Center  
For more information, follow [this instruction](/aws/iam/create-user-with-administrative-access-in-iam-identity-center-en).
- A GitHub repository

## Environment
- Ubuntu 22.04.3 LTS (running on WSL)
- Docker Engine 26.0.0
- Laravel 11

## Deployment Steps
1. [Deploy a Project](#1-deploy-a-project)
2. [Create an OIDC Identity Provider](#2-create-an-oidc-identity-provider)
3. [Create a Role for GitHub OIDC Identity Provider](#3-create-a-role-for-github-oidc-identity-provider)
4. [Add Role to Assume to Repository Secrets](#4-add-role-to-assume-to-repository-secrets)
5. [Create a Task Definition in Your Repository](#5-create-a-task-definition-in-your-repository)
6. [Create a Workflow](#6-create-a-workflow)
7. [Verify the Workflow](#7-verify-the-workflow)

## 1. Deploy a Project
Deploy your project to ECS.  
For details, refer to [this guide](/aws/ecs/deploy-laravel-to-ecs-using-fargate-en).

## 2. Create an OIDC Identity Provider
OpenID Connect (OIDC) is used for authentication with AWS.  
Here's an explanation from [this page](https://docs.github.com/en/actions/how-tos/deploy/deploy-to-third-party-platforms/amazon-elastic-container-service).

> OpenID Connect (OIDC) allows your GitHub Actions workflows to access resources in Amazon Web Services (AWS), without needing to store the AWS credentials as long-lived GitHub secrets.

To create an OIDC identity provider, go to [IAM](https://console.aws.amazon.com/iam/) on the AWS Management Console.  
Click "Identity providers".

![Side bar on IAM](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Side bar on IAM")

Click "Add provider".

![Identity providers](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image2.png "Identity providers")

Select "OpenID Connect" and enter the following values, as provided in [this documentation](https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-aws).

| Provider URL | https://token.actions.githubusercontent.com |
| Audience | sts.amazonaws.com |

![Identity provider creation](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image3.png "Identity provider creation")

## 3. Create a Role for GitHub OIDC Identity Provider
Go to "Role" on [IAM](https://console.aws.amazon.com/iam/) on the AWS Management Console.

![Side bar on IAM](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image4.png "Side bar on IAM")

Click "Create role".

![Roles](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image5.png "Roles")

Select "Web identity".

![Select trusted entity screen1](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image6.png "Select trusted entity screen1")

Fill in the following fields, then click "Next".

| Field | Description |
|-------|-------------|
| Identity provider | The OIDC identity provider you created. |
| Audience | The audience you specified. |
| GitHub organization| Your GitHub user name. |
| GitHub repository | The repository name(s) this role will be used for. Use `*` for all repositories. |
| GitHub branch | The branch name(s) this role will be used for. Use `*` for all branches. |

![Select trusted entity screen2](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image7.png "Select trusted entity screen2")

Click "Next" on the "Add permission" screen.

Enter a role name and description, then click "Create role".

![Final screen](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image8.png "Final screen")

Attach required policies for ECR, ECS and IAM.  
Click the role you just created on [IAM](https://console.aws.amazon.com/iam/) on the AWS Management Console.

Click "Create inline policy".

![Roles](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image9.png "Roles")

Switch to the JSON tab, then enter the following policy.

For ECR:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "ecr:BatchGetImage",
                "ecr:CompleteLayerUpload",
                "ecr:UploadLayerPart",
                "ecr:InitiateLayerUpload",
                "ecr:BatchCheckLayerAvailability",
                "ecr:PutImage"
            ],
            "Resource": "arn:aws:ecr:<Your repository region>:<Your Account ID>:repository/<Your repository name>"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": "ecr:GetAuthorizationToken",
            "Resource": "*"
        }
    ]
}
```

Click "Next", then enter a policy name.  
Click "Create policy".

Repeat the same steps for ECS and IAM.

For ECS:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "ecs:UpdateService",
                "ecs:RegisterTaskDefinition",
                "ecs:DescribeServices"
            ],
            "Resource": [
                "arn:aws:ecs:<Your cluster region>:<Your Account ID>:service/<Your cluster name>/*",
                "arn:aws:ecs:<Your cluster region>:<Your Account ID>:task-definition/<Your task definition name>:*"
            ]
        }
    ]
}
```

For IAM:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "iam:PassRole",
            "Resource": "arn:aws:iam::<Your Account ID>:role/<Your task execution role name>"
        }
    ]
}
```

## 4. Add Role to Assume to Repository Secrets
Go to your GitHub repository and click "Settings".

![GitHub repository](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image10.png "GitHub repository")

Click "Secrets and Variables" > "Actions".

![Settings](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image11.png "Settings")

Click "New repository secret".

![Actions secrets and variables](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image12.png "Actions secrets and variables")

Enter a name (e.g. `ROLE_TO_ASSUME`) and paste your role ARN as the secret value.  
Click "Add secret".

![Secret creation](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image13.png "Secret creation")

## 5. Create a Task Definition in Your Repository
Create a task definition JSON file in your repository.  
For example, save it as `.aws/task-definition.json`.

## 6. Create a Workflow
Go to "Actions" on your GitHub repository.

![GitHub repository](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image14.png "GitHub repository")

Click "New workflow".

![Actions](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image15.png "Actions")

Click "Configure" of "Deploy to Amazon ECS".

![Choose a workflow screen](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image16.png "Choose a workflow screen")

This workflow builds a Docker image and push it to ECR.  
Then it replaces "image" in the task definition with new one and deploys the task definition to ECS.

Update the environment variables with your actual values in the YAML file.

```yml
AWS_REGION: MY_AWS_REGION                   # set this to your preferred AWS region, e.g. us-west-1
ECR_REPOSITORY: MY_ECR_REPOSITORY           # set this to your Amazon ECR repository name
ECS_SERVICE: MY_ECS_SERVICE                 # set this to your Amazon ECS service name
ECS_CLUSTER: MY_ECS_CLUSTER                 # set this to your Amazon ECS cluster name
ECS_TASK_DEFINITION: MY_ECS_TASK_DEFINITION # set this to the path to your Amazon ECS task definition
                                               # file, e.g. .aws/task-definition.json
CONTAINER_NAME: MY_CONTAINER_NAME           # set this to the name of the container in the
                                               # containerDefinitions section of your task definition
```

Add `id-token: write` to `permissions` for OIDC.

```yml
permissions:
  id-token: write # Add
  contents: read
```

Update "Configure AWS credentials" to use `ROLE_TO_ASSUME`.

{% raw %}

```yml
- name: Configure AWS credentials
  uses: aws-actions/configure-aws-credentials@v1
  with:
    role-to-assume: ${{ secrets.ROLE_TO_ASSUME }}  # Add
    aws-region: ${{ env.AWS_REGION }}
```

{% endraw %}

If your `Dockerfile` isn't in the root directory, specify the `-f` option in the `docker build` command.

Example:

```yml
docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG -f ./docker/web/Dockerfile .
```

Enter a file name (e.g. `aws.yml`) and click "Commit changes".

![Workflow creation](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image17.png "Workflow creation")

This workflow will run because it's triggered when changes are pushed to the "main" branch.

```yml
on:
  push:
    branches: [ "main" ]
```

## 7. Verify the Workflow
Go to "Actions" on your GitHub repository.

![GitHub repository](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image14.png "GitHub repository")

If you see a successful run like the following, the workflow has completed successfully.

![Workflow screen](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image18.png "Workflow screen")
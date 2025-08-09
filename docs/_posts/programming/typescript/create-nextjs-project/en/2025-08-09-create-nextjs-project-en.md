---
layout: my-post
title: "Create a Next.js Project"
date: 2025-08-09 00:00:00 +0000
categories: programming typescript
page_name: create-nextjs-project-en
lang: en
---

This article explains how to create a Next.js project using Docker.

## Reference
- [Creating a React App](https://react.dev/learn/creating-a-react-app)

## What is Next.js?
Next.js is a React framework that enables you to create web applications.  
Using the App Router allows you to create a full-stack project.

## Environment
- Ubuntu 22.04.3 LTS (running on WSL)
- Docker Engine 26.0.0
- Next.js 15.4.6

## Creation Steps
1. [Create a Next.js Project](#1-create-a-nextjs-project)
2. [Verify the Next.js Project](#2-verify-the-nextjs-project)

## 1. Create a Next.js Project
Navigate to the directory where you want to create the Next.js project.

Run the following command to start a Node.js Docker container.  
In this example, we use the `node:24.5.0-alpine` Docker image.  
The Node.js version must be `18.18` or later.

```bash
docker run --name nextjs-project --rm -it -w /app -v `pwd`:/app -p 3000:3000 node:2
4.5.0-alpine sh
```

This command runs a Docker container named `nextjs-project` and connects to it using `sh`.  
For more information about the `docker run` command, refer to [this page](/platform/docker/about-docker-commands-en#docker-run).

Inside the container, run the following command to create a Next.js app.

```bash
npx create-next-app@latest
```

Answer the following questions.  
In this example, choose the options shown below.

```bash
What is your project named? … <Your project name>
Would you like to use TypeScript? … Yes
Would you like to use ESLint? … Yes
Would you like to use Tailwind CSS? … Yes
Would you like your code inside a `src/` directory? … No
Would you like to use App Router? (recommended) … Yes
Would you like to use Turbopack for `next dev`? … Yes
Would you like to customize the import alias (`@/*` by default)? … No
```

|Question|Description|
|------|-----------|
|Would you like to use TypeScript?|Whether to use TypeScript. If not, a project will be set up with JavaScript.|
|Would you like to use ESLint?|Whether to use ESLint. ESLint is a tool that finds and fixes problems in JavaScript or TypeScript code.|
|Would you like to use Tailwind CSS?|Whether to use Tailwind CSS. Tailwind CSS is a modern CSS framework.|
|Would you like to use App Router?|Whether to use App Router. App Router is a routing system that allows you to create a full-stack project.|
|Would you like to use Turbopack for `next dev`?|Whether to use Turbopack. Turbopack is an incremental bundler that makes local development much faster.|
|Would you like to customize the import alias (`@/*` by default)|Wether to customize the import alias.|

## 2. Verify the Next.js Project
Run the following commands in the Docker container to launch the local server.

```bash
cd <Your project name>
npm run dev
```

Open `http://localhost:3000` in your browser.  
If you see the following page, the project was successfully created.

![The top page on a Next.js project](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "The top page on a Next.js project")
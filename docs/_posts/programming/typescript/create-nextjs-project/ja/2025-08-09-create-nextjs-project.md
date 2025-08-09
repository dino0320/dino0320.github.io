---
layout: my-post
title: "Next.jsプロジェクトを作成する"
date: 2025-08-09 00:00:00 +0000
categories: programming typescript
page_name: create-nextjs-project
lang: ja
---

この記事では、Dockerを使用してNext.jsプロジェクトを作成する方法を説明します。

## 参考
- [Creating a React App](https://react.dev/learn/creating-a-react-app)

## Next.jsとは？
Next.jsはWebアプリケーションを作成できるReactフレームワークです。  
App Routerというルーティングシステムを使用することで、フルスタックのプロジェクトを作成できます。

## 環境
- Ubuntu 22.04.3 LTS (WSLで起動している)
- Docker Engine 26.0.0
- Next.js 15.4.6

## 作成手順
1. [Next.jsプロジェクトを作成する](#1-nextjsプロジェクトを作成する)
2. [Next.jsプロジェクトを確認する](#2-nextjsプロジェクトを確認する)

## 1. Next.jsプロジェクトを作成する
Next.jsプロジェクトを作成するディレクトリへ移動します。

次のコマンドを実行して、Node.jsのDockerコンテナを起動します。  
この例では、`node:24.5.0-alpine` のDockerイメージを使用します。  
Node.jsのバージョンは `18.18` 以上である必要があります。

```bash
docker run --name nextjs-project --rm -it -w /app -v `pwd`:/app -p 3000:3000 node:2
4.5.0-alpine sh
```

このコマンドは `nextjs-project` という名前のDockerコンテナを起動し、`sh` を使ってそのコンテナに接続します。  
`docker run` コマンドについての詳細は[こちらのページ](/platform/docker/about-docker-commands#docker-run)を参照してください。

コンテナ内で次のコマンドを実行してNext.jsアプリを作成します。

```bash
npx create-next-app@latest
```

以下の質問に答えます。  
今回は次のように選択します。

```bash
What is your project named? … <プロジェクト名>
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
|Would you like to use TypeScript?|TypeScriptを使用するかどうか。使用しない場合はJavaScriptでプロジェクトが作成されます。|
|Would you like to use ESLint?|ESLintを使用するかどうか。ESLintはJavaScriptやTypeScriptのコード内の問題を検出・修正するツールです。|
|Would you like to use Tailwind CSS?|Tailwind CSSを使用するかどうか。Tailwind CSSはモダンなCSSフレームワークです。|
|Would you like to use App Router?|App Routerを使用するかどうか。App Routerはルーティングシステムで、フルスタックのプロジェクトを作成できます。|
|Would you like to use Turbopack for `next dev`?|Turbopackを使用するかどうか。Turbopackは増分バンドラーで、ローカル開発時のリアルタイムビルドを高速化します。|
|Would you like to customize the import alias (`@/*` by default)|インポートエイリアスをカスタマイズするかどうか。|

## 2. Next.jsプロジェクトを確認する
Dockerコンテナ内で次のコマンドを実行してローカルサーバーを起動します。

```bash
cd <プロジェクト名>
npm run dev
```

ブラウザで `http://localhost:3000` にアクセスします。
次のページが表示されれば、プロジェクトは正常に作成されています。

![プロジェクトのトップページ](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "プロジェクトのトップページ")
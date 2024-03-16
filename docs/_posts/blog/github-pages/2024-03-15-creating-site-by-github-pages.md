---
layout: my-post
title: "GitHub Pagesでサイトを作る"
date: 2024-03-15 17:00:00 -0000
categories: blog github-pages
---

GitHub Pagesを利用してウェブサイトを公開します。

## GitHub Pagesとは
GitHubのリポジトリから自動で静的サイトを公開してくれるサービスです。  
HTMLやMarkdownで記載されたファイルからウェブサイト作成します。  
[GitHub Pagesについて](https://docs.github.com/ja/pages/getting-started-with-github-pages/about-github-pages)

## サイト作成の流れ
1. [GitHubのアカウントを作成する。](#1-githubのアカウントを作成する)
2. [GitHubのリポジトリを作成する。](#2-githubのリポジトリを作成する)
3. [リポジトリの設定を変更する。](#3-リポジトリの設定を変更する)
4. [サイトを確認する。](#4-サイトを確認する)

[GitHub Pages サイトを作成する](https://docs.github.com/ja/pages/getting-started-with-github-pages/creating-a-github-pages-site)

### 1. GitHubのアカウントを作成する
GitHubのアカウントを持ってない場合、GitHubのアカウントを作成します。   
持っている場合はサインインして次のステップに進みます。
1. [GitHub](https://github.co.jp/)にアクセスする。
2. 「[サインアップ](https://github.com/signup)」または「[GitHubに登録する](https://github.com/signup)」をクリックする。  
![GitHubサインアップボタン](/assets/images/blog/github-pages/2024-03-15-creating-site-by-github-pages/image1.png "GitHubサインアップボタン")
3. 案内に従ってアカウントを作成する。

### 2. GitHubのリポジトリを作成する
GitHubのリポジトリを作成します。
1. [GitHub](https://github.com/)にアクセスする。
2. 「[New](https://github.com/new)」をクリックする。  
![GitHubリポジトリ作成ページにアクセスするボタン](/assets/images/blog/github-pages/2024-03-15-creating-site-by-github-pages/image2.png "GitHubリポジトリ作成ページにアクセスするボタン")
3. 以下の項目を確認する。

|項目|説明|
|----|----|
|Owner|リポジトリを所有するアカウントを選択します。|
|Repository name|リポジトリ名です。リポジトリ名は `<アカウント名>.github.io` にする必要があります。アカウント名は全て小文字にして入力します。|
|Description|リポジトリの説明です。任意の説明を入力します。|
|リポジトリの可視性|「Public」か「Private」を選択します。アカウントがGitHub Freeの場合、「Public」を選択する必要があります。|
|Add a README file|`README.md` を追加するかどうかです。GitHub Pagesでは `index.html` または `index.md`、`README.md` がトップページになります。今回はチェックを入れます。|

4. 「Create repository」をクリックする。  
![GitHubリポジトリ作成ボタン](/assets/images/blog/github-pages/2024-03-15-creating-site-by-github-pages/image3.png "GitHubリポジトリ作成ボタン")

### 3. リポジトリの設定を変更する
サイトを公開するためにリポジトリの設定を変更します。
1. [GitHub](https://github.com/)にアクセスする。
2. 作成したGitHubリポジトリのページにアクセスする。  
![GitHubリポジトリのページにアクセスするリンク](/assets/images/blog/github-pages/2024-03-15-creating-site-by-github-pages/image4.png "GitHubリポジトリのページにアクセスするリンク")
3. 「Settings」をクリックする。  
![Settingボタン](/assets/images/blog/github-pages/2024-03-15-creating-site-by-github-pages/image5.png "Settingボタン")
4. 「Pages」をクリックする。  
![Pagesボタン](/assets/images/blog/github-pages/2024-03-15-creating-site-by-github-pages/image6.png "Pagesボタン")
5. 「Build and deployment」の以下の項目を確認する。

|項目|説明|
|----|----|
|Source|サイトの公開元です。今回は「Deploy from a branch」を選択します。特定のブランチにプッシュされたときにサイトを公開します。|
|Branch|ブランチとルートフォルダを選択します。選択したブランチにプッシュすると、選択したルートフォルダ内の変更がサイトに公開されます。ルートフォルダは `/` または `/docs` を指定できます。今回は `/` を選択します。|

### 4. サイトを確認する
公開したサイトを確認します。
1. [GitHub](https://github.com/)にアクセスする。
2. 作成したGitHubリポジトリのページにアクセスする。  
![GitHubリポジトリのページにアクセスするリンク](/assets/images/blog/github-pages/2024-03-15-creating-site-by-github-pages/image4.png "GitHubリポジトリのページにアクセスするリンク")
3. 「Settings」をクリックする。  
![Settingボタン](/assets/images/blog/github-pages/2024-03-15-creating-site-by-github-pages/image5.png "Settingボタン")
4. 「Pages」をクリックする。  
![Pagesボタン](/assets/images/blog/github-pages/2024-03-15-creating-site-by-github-pages/image6.png "Pagesボタン")
5. 「Visit site」をクリックする。  
![Visit siteボタン](/assets/images/blog/github-pages/2024-03-15-creating-site-by-github-pages/image7.png "Visit siteボタン")  
トップページ(`README.md`)が確認できます。
---
layout: my-post
title: "Jekyllを使用してGitHub Pagesのサイトを作る"
date: 2024-03-20 00:00:00 +0000
categories: blog github-pages
page_name: creating-site-by-github-pages-with-jekyll
lang: ja
image: /assets/images/blog/github-pages/creating-site-by-github-pages-with-jekyll/image7.png
---

Jekyllを使用してGitHub Pagesのサイトを作成します。  
Jekyllは静的サイトジェネレーターです。GitHub PagesではデフォルトでJekyllを使用してサイトを作成しています。  

![サムネイル](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image7.png "サムネイル")

## 環境
- Windows 10 64ビット
- Git for Windows 2.44.0
- Ruby 3.2.3
- Jekyll 4.3.3

## 前提
- Gitをインストールしている。
- GitHubのアカウントを持っている。

## サイト作成の流れ
1. [Jekyllをインストールする。](#1-jekyllをインストールする)
2. [GitHub Pagesでサイトを作る。](#2-github-pagesでサイトを作る)
3. [GitHubリポジトリをクローンする。](#3-githubリポジトリをクローンする)
4. [Jekyllを使用してサイトを作る。](#4-jekyllを使用してサイトを作る)
5. [ローカルでサイトを確認する。](#5-ローカルでサイトを確認する)
6. [変更をコミット・プッシュする。](#6-変更をコミットプッシュする)
7. [リポジトリの設定を変更する。](#7-リポジトリの設定を変更する)
8. [サイトを確認する。](#8-サイトを確認する)

以下のページを参考にしています。  
[Jekyll を使用して GitHub Pages サイトを作成する](https://docs.github.com/ja/pages/setting-up-a-github-pages-site-with-jekyll/creating-a-github-pages-site-with-jekyll)

## 1. Jekyllをインストールする
Jekyllのインストールについては[こちら](/programming/ruby/installing-jekyll-on-windows)をご覧ください。  

## 2. GitHub Pagesでサイトを作る
GitHub Pagesのサイト作成については[こちら](/blog/github-pages/creating-site-by-github-pages)をご覧ください。  

## 3. GitHubリポジトリをクローンする
Jekyllを使用してサイトを作成します。  
1. [GitHub](https://github.com/)にアクセスする。  
サインインしていない場合、サインインします。
2. 作成したGitHubリポジトリのページにアクセスする。  
![GitHubリポジトリのページにアクセスするリンク](/assets/images/blog/github-pages/creating-site-by-github-pages-with-jekyll/image1.png "GitHubリポジトリのページにアクセスするリンク")
3. 「Code」をクリックする。
4. クローン用のURLをコピーする。  
![クローン用のURL](/assets/images/blog/github-pages/creating-site-by-github-pages-with-jekyll/image2.png "クローン用のURL")
5. GitHubリポジトリをクローンする。    
コマンドプロンプトかPowerShellを開き、以下を実行します。  
`<リポジトリをクローンするディレクトリのパス>` は任意のパスです。  
`https://github.com/<アカウント名>/<リポジトリ名>.git` はコピーしたクローン用のURLです。
```
> cd <リポジトリをクローンするディレクトリのパス>
> git clone https://github.com/<アカウント名>/<リポジトリ名>.git
```

## 4. Jekyllを使用してサイトを作る
1. リポジトリのディレクトリに移動する。  
コマンドプロンプトかPowerShellを開き、以下を実行します。  
`<リポジトリのディレクトリのパス>` は `<リポジトリをクローンするディレクトリのパス>/<リポジトリ名>` です。
```
> cd <リポジトリのディレクトリのパス>
```
2. `README.md` を削除する。  
リポジトリのルートディレクトリに `README.md` がある場合、`README.md` を削除します。  
以下を実行します。
```
> git rm README.md
```
3. `docs` ディレクトリを作成して移動する。  
サイトを作成するディレクトリを作成して移動します。以下を実行します。
```
> mkdir docs
> cd docs
```
4. Jekyllでサイトを作成する。  
現在のディレクトリ(`docs` ディレクトリ)にサイトを作成します。以下を実行します。
```
> jekyll new --skip-bundle .
```
5. `Gemfile` を変更する。  
作成された `Gemfile` をテキストエディターで以下のように変更します。  
GitHub Pagesを使う場合は `github-pages` が必要なので、次の行をコメントから戻します。  
```
gem "github-pages", group: :jekyll_plugins
```
上記の行に `, "~> <github-pagesのバージョン>"` を追加します。`<github-pagesのバージョン>` には[Dependency versions](https://pages.github.com/versions.json)の `github-pages` のバージョンを入力します。  
```
gem "github-pages", "~> <github-pagesのバージョン>", group: :jekyll_plugins
```
`github-pages` に含まれるので、`jekyll` を定義する行をコメントアウトします。
```
#gem "jekyll", "~> <jekyllのバージョン>"
```
6. `_config.yml` を変更する。  
作成された `_config.yml` をテキストエディターで変更します。  
以下の項目を確認して変更します。サイトで表示したくない場合は項目ごとコメントアウトします。

|項目|説明|
|----|----|
|title|サイトのタイトルを入力します。|
|description|サイトの説明を入力します。|
|twitter_username|Xのアカウント名を入力します。|
|github_username|GitHubのアカウント名を入力します。|

7. gemをインストールする。  
gemをインストールするため、以下を実行します。
```
> bundle install
```

## 5. ローカルでサイトを確認する
1. ローカルでサーバーを立ち上げる。  
コマンドプロンプトかPowerShellを開き、以下を実行します。
```
> cd <リポジトリのディレクトリのパス>/docs
> bundle add webrick # Rubyのバージョンが3.0以降の場合に実行します。
> bundle exec jekyll serve
```
2. http://localhost:4000 にアクセスする。  
以下のようにトップページが表示されます。  
サイトのテーマはデフォルトで[minima](https://github.com/jekyll/minima)が使用されています。  
「Welcome to Jekyll!」は `jekyll new` で作られたページです。  
![トップページ](/assets/images/blog/github-pages/creating-site-by-github-pages-with-jekyll/image3.png "トップページ")

## 6. 変更をコミット・プッシュする
1. `.gitignore` を変更する。  
GitHub Pagesは `Gemfile` と `Gemfile.lock` を見てインストールしません。  
そのため、作成された `.gitignore` に以下の行を追加します。
```
Gemfile
Gemfile.lock
```
2. コミット・プッシュする。  
コマンドプロンプトかPowerShellを開き、以下を実行します。
```
> cd <リポジトリのディレクトリのパス>
> git add docs/
> git commit -m "Jekyllで作成したサイトを追加"
> git push origin main
```

## 7. リポジトリの設定を変更する
リポジトリの設定を変更します。
1. [GitHub](https://github.com/)にアクセスする。
2. 作成したGitHubリポジトリのページにアクセスする。  
![GitHubリポジトリのページにアクセスするリンク](/assets/images/blog/github-pages/creating-site-by-github-pages-with-jekyll/image1.png "GitHubリポジトリのページにアクセスするリンク")
3. 「Settings」をクリックする。  
![Settingボタン](/assets/images/blog/github-pages/creating-site-by-github-pages-with-jekyll/image4.png "Settingボタン")
4. 「Pages」をクリックする。  
![Pagesボタン](/assets/images/blog/github-pages/creating-site-by-github-pages-with-jekyll/image5.png "Pagesボタン")
5. 「Build and deployment」の「Branch」のルートフォルダを変更する。  
「/docs」を選択します。

## 8. サイトを確認する
公開したサイトを確認します。
1. [GitHub](https://github.com/)にアクセスする。
2. 作成したGitHubリポジトリのページにアクセスする。  
![GitHubリポジトリのページにアクセスするリンク](/assets/images/blog/github-pages/creating-site-by-github-pages-with-jekyll/image1.png "GitHubリポジトリのページにアクセスするリンク")
3. 「Settings」をクリックする。  
![Settingボタン](/assets/images/blog/github-pages/creating-site-by-github-pages-with-jekyll/image4.png "Settingボタン")
4. 「Pages」をクリックする。  
![Pagesボタン](/assets/images/blog/github-pages/creating-site-by-github-pages-with-jekyll/image5.png "Pagesボタン")
5. 「Visit site」をクリックする。  
![Visit siteボタン](/assets/images/blog/github-pages/creating-site-by-github-pages-with-jekyll/image6.png "Visit siteボタン")  
トップページが確認できます。  
![トップページ](/assets/images/blog/github-pages/creating-site-by-github-pages-with-jekyll/image3.png "トップページ")
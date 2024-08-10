---
layout: my-post
title: "GitのfileModeについて"
date: 2024-06-10 00:00:00 +0000
categories: version-control git
title_eng: about-filemode-in-git-config
lang: ja
---

Gitのconfigの `core.fileMode` について調査しました。

## 参考ページ
- [git config](https://git-scm.com/docs/git-config)

## 環境
- Windows 10 64ビット
- Ubuntu 22.04.3 LTS (WSLで起動している)
- git version 2.34.1

## core.fileModeとは
Gitのconfigの `core.fileMode` は、実行可能なファイルかどうかをGitに伝えるための設定です。  
値を `true` にすると、ファイルの実行権限を変更したときにその変更を教えてくれます。  
`false` にすると、その変更を無視するようになります。  
デフォルト値は `true` です。

Windowsなどのファイルシステムでは実行権限という概念がないため、実行可能なファイルをチェックアウトしたときなどに実行権限が失われます。  
そのようなファイルシステムでリポジトリを操作するときは、`core.fileMode` を `false` にするといいかもしれません。

## core.fileModeを設定する
以下のコマンドで `core.fileMode` を設定できます。  
`<値>` には `true` または `false` を指定します。

```
git config core.filemode <値>
```

以下のコマンドで `core.fileMode` の値を確認できます。

```
git config core.filemode
```

## core.fileModeを確認する
`core.fileMode` を確認します。  
Ubuntu上で行っています。

`core.fileMode` を `false` にします。  

```bash
$ git config core.filemode false
```

実行権限のない適当なファイルに実行権限を与えます。  
今回は適当なファイルを `hoge.sh` とします。

```bash
$ chmod 755 hoge.sh
```

権限を確認すると、実行権限(x)が与えられていることが分かります。

```bash
$ ls -l hoge.sh
-rwxr-xr-x ～略～ hoge.sh
```

`core.fileMode` が `false` なので、`git status` を実行しても変更が確認できません。

```bash
$ git status    # 変更したファイルは表示されないはず
```

`core.fileMode` を `true` にします。 

```bash
$ git config core.filemode true
```

`git status` を実行すると、ファイルの変更が確認できます。

```bash
$ git status

～略～

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   hoge.sh

～略～
```
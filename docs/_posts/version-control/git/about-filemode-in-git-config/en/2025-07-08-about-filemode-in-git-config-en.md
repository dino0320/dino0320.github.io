---
layout: my-post
title: "About Git’s fileMode Setting"
date: 2025-07-08 00:00:00 +0000
categories: version-control git
page_name: about-filemode-in-git-config-en
lang: en
---

I investigated the `core.fileMode` setting in Git's config.

## Reference
- [git-config Documentation - Git](https://git-scm.com/docs/git-config)

## Environment
- Windows 10 64-bit
- Ubuntu 22.04.3 LTS (running via WSL)
- git version 2.34.1

## What is core.fileMode?
The `core.fileMode` setting in Git’s configuration determines whether Git tracks changes to a file’s executable permission (i.e., whether a file is marked as executable).

When set to `true`, Git will detect and report changes to file permissions (e.g., adding or removing the executable flag).

When set to `false`, Git will ignore permission changes.

The default value is `true`.

Some filesystems—such as those used on Windows—do not support Unix-style executable permissions.  
As a result, executable flags may be lost when checking out files.  
If you're working in such an environment, setting `core.fileMode` to `false` might be helpful.

## Setting core.fileMode
You can configure `core.fileMode` using the following command, replacing `<value>` with `true` or `false`:

```
git config core.filemode <value>
```

To check the current value:

```
git config core.filemode
```

## Testing core.fileMode
Let’s test the effect of `core.fileMode`.  
This example is performed on Ubuntu.

First, set `core.fileMode` to `false`: 

```bash
$ git config core.filemode false
```

Next, give executable permissions to a file that currently doesn't have them.
We’ll use a sample file named `hoge.sh`:

```bash
$ chmod 755 hoge.sh
```

Verify that the file now has execute permission (x):

```bash
$ ls -l hoge.sh
-rwxr-xr-x ... hoge.sh
```

Since `core.fileMode` is set to `false`, running `git status` should not show any changes:

```bash
$ git status    # The modified file should not be listed
```

Now, set `core.fileMode` back to `true`:

```bash
$ git config core.filemode true
```

Run `git status` again, and you’ll see that Git detects the change in file permissions:

```bash
$ git status

...

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   hoge.sh

...
```
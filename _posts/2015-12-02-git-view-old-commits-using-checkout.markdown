---
layout: post
title: git如何checkout历史提交
category: Document
tags: git
date: 2015-12-02 15:12:32
published: true
summary: 应用场景是，看大师们的代码库，比如我想看看大牛们是如何一点点把牛气冲天的linux内核写出来的。
image: pirates.svg
comment: true
---

## 需求

我想查看某个git代码版本库的历史提交记录，并看某特定提交的代码，如何做？


## 解决办法

git checkout 可以办到。

### Step 1： 列出所有的commit

```
root$ git log
```

找到需要查看的提交的hash值，比如这次提交的`c3f1a81a5dde6ccea1ec71c446cf96502b0a778a`。

```
commit c3f1a81a5dde6ccea1ec71c446cf96502b0a778a
Author: Joshua Liu <liuchaozhenyu@gmail.com>
Date:   Mon Nov 30 17:29:42 2015 +0800

    open doc|xls in terminal

```

### Step 2：checkout

```
git checkout <commit hash id>
```

就这么简单。

## 讨论

### 1. git log

     git 命令      |                  说明
-------------------|------------------------------------------
 git log           | 不加参数将按commit时间顺序列出repo中所有的代码提交。
 git log -p [-2]   | 参数将显示提交的代码变动情况。默认显示所有提交，加`-2`参数后只显示最后两次提交。
 git log --stat    | 列举提交历史的时候，附带简单的变更信息，比如多少文件改变了，文件添加了或减少了多少代码行
 git log --pretty  | 使log结果更好看


```
$ git log --pretty=oneline
ca82a6dff817ec66f44342007202690a93763949 changed the version number
085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7 removed unnecessary test
a11bef06a3f659402fe7563abf99ad00de2209e6 first commit
```


```
$ git log --pretty=format:"%h - %an, %ar : %s"
ca82a6d - Scott Chacon, 6 years ago : changed the version number
085bb3b - Scott Chacon, 6 years ago : removed unnecessary test
a11bef0 - Scott Chacon, 6 years ago : first commit
```

参考：

> https://git-scm.com/book/en/v2/Git-Basics-Viewing-the-Commit-History

### 2. git checkout

git checkout 主要有三种用途:

1. 检出分支（branch）

    ```
    $ git checkout <branch>
    ```
2. 检出文件

    ```
    $ git checkout <commit> <file>
    ```
3. 检出提交(commit or tag)

    ```
    $ git checkout <commit>
    ```
    注意此时的“状态”是未附着在master分支上的，且是以只读形式存在的，因为已提交的内容是不容许修改的。

参考：

> https://www.atlassian.com/git/tutorials/viewing-old-commits/


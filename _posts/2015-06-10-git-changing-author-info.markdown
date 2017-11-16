---
layout: post
title: 使用脚本修改git提交后的user-email信息
category: Document
tags: git
year: 2015
month: 06
day: 10
published: true
summary: Git提交后信息修改。
image: pirates.svg
comment: true
---

## 修改提交者信息

要修改已经提交commit的姓名或者email信息，你必须重写git仓库的整个历史记录。

_警告：这样操作将破坏你的仓库历史。如果和别人协作一个仓库的话，重写提交历史将非常糟糕。只有在救急的情况下才能这么做_

## 使用脚本来修改Git历史记录
----------

github 创建了一个脚本可以改变已提交的任意作者名和email信息。

_注意：运行词脚本将重写整个仓库的历史记录。完成修改后，任何人想fork或者clone必须fetch重写后的历史记录并且要把任何本地修改rebase到重写的历史记录里。_

在运行本脚本前，你需要为脚本提供：

- 需要修改的作者/提交者的旧的email地址。
- 正确的姓名和email信息。

执行步骤：

1. 打开终端
2. 创建新的，bare复制要修改的仓库：

    ```
    $ git clone --bare https://github.com/usr/repo.git
    $ cd repo.git
    ```
3. 复制并粘贴以下脚本，替换以下变量的信息为正确的email和姓名“
    - OLD_EMAIL
    - CORRECT_NAME
    - CORRECT_EMAIL

    ```sh
    #!/bin/sh

    git filter-branch --env-filter '
    OLD_EMAIL="your-old-email@example.com"
    CORRECT_NAME="Your Correct Name"
    CORRECT_EMAIL="your-correct-email@example.com"

    if [ "$GIT_COMMITTER_EMAIL" = "$OLD_EMAIL" ]
    then
        export GIT_COMMITTER_NAME="$CORRECT_NAME"
        export GIT_COMMITTER_EMAIL="$CORRECT_EMAIL"
    fi
    if [ "$GIT_AUTHOR_EMAIL" = "$OLD_EMAIL" ]
    then
        export GIT_AUTHOR_NAME = "$CORRECT_NAME"
        export GIT_AUTHOR_EMAIL = "$CORRECT_EMAIL"
    fi
    ' --tag-name-filter cat -- --branches --tags
    ```
4. 回车执行该脚本
5. 查看新的仓库历史记录是否有错误
6. push 正确的历史记录到远程仓库：

    ```
    $ git push --force --tags origin 'refs/heads/*'
    ```
7. 清楚临时仓库

    ```
    $ cd ..
    $ rm -rf repo.git
    ```



> 参考：https://help.github.com/articles/changing-author-info/

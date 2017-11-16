---
layout: post
title: linux 终端显示 git 分支名
category: Document
tags: git macos linux
date: 2016-12-14 20:12:02
published: true
summary: 
image: pirates.svg
comment: true 
---

MacOS

```sh
$ vim ~/.bash_profile
```

写入以下内容：

```sh
# Git branch in prompt
parse_git_branch() {
    git branch 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/ (\1)/'
}

export PS1="\u@\h \W\[\033[32m\]\$(parse_git_branch)\[\033[00m\] $ "
```

加载配置：

```sh
source ~/.bash_profile
```

就可以看到效果。

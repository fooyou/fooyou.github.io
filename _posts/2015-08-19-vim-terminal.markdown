---
layout: post
title: Vim工作时和Terminal的切换
category: Document
tags: vim terminal
date: 2015-08-19 16:55:03
published: true
summary:
image: pirates.svg
comment: true
---

有很多方法，我最喜欢这个：

vim工作环境中，启动Terminal：

```
:sh
```

返回Terminal在当前vim工作目录，再次切回vim使用`Ctrl + d`

```
$ <Ctrl + d>
```

> 参考：http://stackoverflow.com/questions/1236563/how-to-run-a-terminal-inside-of-vim
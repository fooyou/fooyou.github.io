---
layout: post
title: OSX library not found for -lintl
category: Document
tags: maxos
date: 2017-10-25 14:10:35
published: true
summary: 二次开发一个开源库，在 MAC 上出现找不到库 libintl，这是什么鬼？！
image: pirates.svg
comment: true
---

这两天二次开发一个开源库，原来的库没有第三方依赖，后添加一些功能，需要用到 fribidi，通过 brew 安装依赖库，通过 find_package 导入 CMakeList.txt 都无问题，但是 make 的时候出现这个问题：

```
error ld: library not found for -lintl
```

不知这是什么鬼，google 了一下发现它是 gettext 库里的一个子库，而我机器上已经安装了 gettext，所以直接建立链接：

```
$ ln -s /usr/local/Cellar/gettext/0.19.8.1/lib/libintl.* /usr/local/lib/
```

编译成功！

Mark 一下！

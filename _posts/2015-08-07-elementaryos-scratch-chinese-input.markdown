---
layout: post
title: elementary OS Freya中文输入的解决方法
category: Document
tags: linux elementaryos
date: 2015-08-07
published: true
summary: 用这个系统感觉不错，scratch取代了gedit，但不能中文输入……有个GTK_IM_MODULE很容易解决
mathjax: false
highlight: true
---

据说使用iBus的能输入中文，但我用的是fcitx不能输入中文。

有哥们说只要指定`GTK_IM_MODULE='xim'`然后启动 scratch-text-editor就行了，果然好用。

两种执行方式：

一、终端启动：

```bash
$ GTK_IM_MODULE="xim" scratch-text-editor
```

二、修改scratch.desktop

```bash
$ vi /usr/share/applications/scratch-text-editor.desktop
```

修改`Exec=`这行为：

```bash
Exec=env GTK_IM_MODULE=xim scratch-text-editor %U
```

OK, Enjoy!!

---
layout: post
title: Elementary OS如何在ternminal中打开文件浏览器
category: Document
tags: linux
date: 2015-11-13 15:37:10
published: true
summary: linux上总要使用Terminal和file explorer，在ubuntu上安装"Open in terminal"后，可以方便切换，但eos上如何呢？
image: pirates.svg
comment: true
latex: false
---

## Ubuntu上

**Nautilus（文件浏览器的名字） --> Terminal（当前目录的）：**

安装个nautilis-open-terminal之类的就行了，从文件浏览器右键打开终端就可以了。

**Terminal --> Nautilus:**

在终端输入以下就可以了（不要忘记后面的那个点和nautilus还有个空格）：

```
$ nautilus .
```

## Elementary OS上：

这一招到E-OS上不好用了，因为文件浏览器换成了pantheon-files，所以得用她来替换nautilus

```
$ pantheon-files .
```

但已经习惯了ubuntu的nautilus命令，所以别名一下

```
$ vim ~/.bashrc
```

在里面加上alias：

```
alias nautilus="pantheon-files"
```

使其立即生效

```
$ source ~/.bashrc
```

OK，可以自由切换了。


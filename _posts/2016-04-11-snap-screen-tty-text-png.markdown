---
layout: post
title: linux tty 终端的截屏
category: Document
tags: linux
date: 2016-04-11 14:04:12
published: true
summary: 本文记录 ubuntu 系统下截取 tty 屏幕的方法和相关原理。
image: pirates.svg
comment: true
---

为了更有效率码农们开始时不时回归到 tty 终端工作，但有时需要在 tty 终端截屏...什么？使用 iPhone 拍照？别扯了，这不是在侮辱码农吗。所以这篇文档就诞生了。


你想马上就能开始截屏，而不想听听原理？好吧，稍后我再说原理，先告诉你怎么做。


**Q：什么是 tty 终端？**


A： tty 是 TeleTYpewritter 的缩写，1960 年代为帮助视力和听力受损的人们使用电话，发明了 TeleTYpeWritter，于是就有了 tty 。


在 Unix / Linux / BSD 系统上， ttys 是用户可以登录的物理终端，现在的 *nixe 系统的终端分为物理（physical）tty 和虚拟（virtual）vty 终端，无论操作系统有无图形界面，在 Unix 或者类 Unix 系统中都有 tty 终端。


不同的 Linux 版本有着数量不同的 tty，大多数的版本有 5～7个物理 tty。在某些版本中，可通过修改 /etc/inittab 添加其个数。


**Q：我用的是 ubuntu 桌面系统，如何切换到 tty？**


A: Ctrl + Alt + F1 (F1 ~ F6)

进入 tty 并登录后，你可以输入 `tty` 来查看信息

```
$ tty
/dev/tty1
```

如果你使用的是类似 mlterm 等的 GUI 终端，那么输出可能会是：


```
$ fbterm
$ tty
/dev/pts/10
```

好了，说了一点 tty 的基本的知识，接下来将说明如何：

1. 把纯文本 tty 屏幕导出成 .txt 文件
2. 把 tty 终端的内容映射为 JPG 或者 PNG 文件

## 把纯文本 tty 屏幕导出成 .txt 文件

如果你使用 tty 命令看到的是 /dev/tty*，那么就可以使用这种方法，把 tty 界面导出为 txt 文件，使用 cat 命令就可以（注意是管理员权限）：

```
$ cat /dev/vcs1 > tty1_text_screenshot.txt
$ less tty1_text_screenshot.txt
```

cat 命令把控制台的内容 dump 到了文件中，然而这样 dump 出的文件就像文本内部的接口结，大部分是不可读的，但可以使用 less 或者 more 命令来读取。

## 把 tty 终端的内容映射为 JPG 或者 PNG 文件

需要安装第三方插件：[snapscreenshot](http://bisqwit.iki.fi/source/snapscreenshot.html)， 或者 [fbgrab]()

我用的是 fbgrab 所以就以其为例说明使用过程：

下载源码，编译，安装后，就可以在 tty 中使用了，文档上说直接：

```
$ fbgrab gb.png
```

就可以把 framebuffer 转化成 png 图像，可我试过不好用，后来自己转换一下就好用了，写个脚本取个别名：


```
$ cp /dev/fb0 /tmp/xxxx.dump
$ fbgrab -w 1440 -h 900 -b 32 -f /tmp/xxxx.dump fb.png
$ fim fb.png
```

好了，好用就行。

```

      \o/   .------------------------------------.
       |     我使用的是正常的 tty 终端，怎么截取？
      / \   '------------------------------------'

                                   .----------------------.      (\_/)
                                    可以截取成 text 文本。       (O.o)
                                   '----------------------'      (> <)

```

参考：

> http://www.pc-freak.net/blog/how-to-make-screenshot-in-devtty-console-on-gnu-linux-taking-picture-jpeg-png-snapshot-of-text-console-in-systems-without-graphical-environment/

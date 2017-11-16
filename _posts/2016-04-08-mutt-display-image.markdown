---
layout: post
title: mutt 在 tty 下显示图片
category: Document
tags: mutt linux
date: 2016-04-08 16:04:43
published: true
summary: Google 了一大圈，终于能在 tty 终端下显示 mutt 邮件中的图片了。
image: pirates.svg
comment: true
---

最终搞定的是使用工具 [fim(fbi-improved)](http://www.nongnu.org/fbi-improved/)，原先的 fbi 在 tty 下也是可以显示图片的，但不如 fim 智能，fim会根据当前的环境自动切换视频模式（linux frame buffer, X/Xorg SDL lib, X/Xorg Imlib2 lib, ASCII mode aalib)。

fbi 为啥不能满足我的要求？在 tty 下，我会使用 tmux 做多任务管理，而 tmux 本身就是一个虚拟的 FB，所以使用 fbi 时，它会报错，而 fim 可以智能判断当前环境并选择能工作的视频模式，所以在 tmux 下 fim 可以正常工作，非常棒！！

w3m 在不使用外部图片浏览器显示图片时也和 fbi 一样，有哥们把 w3mimagdisplay 重新封装了一下，参见 `http://blog.z3bra.org/2014/01/images-in-terminal.html`，但在 virtual fb 下 w3mimagdisplay 同样不好用。

看到 fim 的智能选择视频模式后，立马安装：

```
$ sudo apt-get install fim
```

Oh， Shit! 没有源，只要自己编译了，于是下了安装包[fim-0.5-rc2.tar.gz](http://savannah.spinellicreations.com/fbi-improved/fim-0.5-rc2.tar.gz)，发现更新还很新，项目一直在维护。

根据 Readme 说明配置安装

```
$ sudo apt-get install automake autoconf libtool
$ sudo apt-get install libreadline-dev libexif-dev
$ sudo apt-get install libjpeg-dev libpng-dev libtiff-dev libgif-dev
$ sudo apt-get install libsdl1.2-dev libaa1-dev
$ sudo apt-get install libpoppler-dev libdjvulibre-dev libspectre-dev
$ sudo apt-get install libarchive-dev
$ cd fim-0.5-rc2
$ ./configure --enable-sdl --enable-aa
$ make
$ sudo make install
```

为什么没有 `--enable-popplar` 因为 popplar 版本更新太快还不稳定，所以出现了编译错误，那索性就不用它。

好了，修改 mutt 的附件配置文件 ~/.mailcap

```
image/*; fim %s; /dev/null
```

去 mutt 里找个有图片附件的邮箱看一下吧！图片出来了，有没有亮瞎眼！



---
layout: post
title: w3m文本终端浏览器
category: Document
tags: linux
date: 2015-10-20 09:13:58
published: true
summary: 突发奇想要在终端使用浏览器，这样在看新闻时就不怕广告图片什么的乱飞了，可以专注在文字阅读上，但若是在tty上呢？
image: pirates.svg
comment: true
latex: false
---

十年前的大学时代，对于刚接触计算机的我来说，所知道的仅仅是有个叫Windows的系统，花几千块配个512M内存和独立显卡就可以流畅的玩魔兽，用电脑还可以聊QQ，看电影。Windows XP——一个观看世界的窗口。学校实验室里牛气冲天的Matlab，AutoCAD全都运行在那个窗口里，这么牛的窗口就这么突然来到我的学生时代，真的不知道这个怪胎从哪里来，都会干什么？

后来开始学编程课先学VB，尼玛，考试分挺高，但是自己知道对这种怪胎缺少认同感，怎么着自己就能写程序了；后来又学了C++，草尼玛，考试刚及格，完全不知道这怪胎是嘛东西，再后来开始学汇编语言，之前的VB，C++完全断片，慢慢的汇编语言里讲到了一些计算机的基本原理，后来才慢慢理解编程是怎么个过程，然后茅塞顿开，原来这所有的怪胎是这么生出来的。原来之前还有DOS，后来，什么unix，linux，freeBCD，草泥马Windows，后来当了程序员才知道Windows坑我这样的人真是不浅，原来真正能反映计算机操作系统发展历史的只有开源操作系统，尤其如果你是一个想让计算机帮助完成某些任务的人（码农）。

操作系统是开源还是封闭好从它们诞生的时候都有人在激烈辩论，除去这些没有意义的辩论（对我而言），我毫不犹豫的站在了开源阵营，原因无他，我是个很平凡的码农，三个臭皮匠顶个诸葛亮的道理还是懂得的。封闭系统就像高门府邸，看起来富丽堂皇，但谁都知道其内里的龌龊，而开源系统就像八大胡同，热闹，繁华，谁家的那点龌龊事都在众人面前一览无余，从而大家共荣，而码农门就是这八大胡同的皮条客。^_^

下面进入正体：

## 终端

Console，Terminal 这是计算机祖师爷门留下的东西，移动操作系统都把它给阉割了（其实都有，只是没释放出来），但终端是什么，打个不恰当的比方，这个东西有点像操作系统的家谱，现在的很多人都不用这东西，但对码农来说这是宝贝的不能在宝贝的东西。和计算机打交道时，键盘是给码农用的，鼠标是给用户用的。

那么现在的终端除了能运行各种命令编写和调试各种代码外，若是能浏览网页进行检索就酷比了，就可以摆脱一边vim一边chrom google了。

浏览网站（纯文字的）比如 lynx，w3m, 等等，哇哦，多么干净的阅读世界，快捷的操作。

等等，如果切换到tty下呢，无法显示中文，有个骨灰级的工具叫[zhcon](http://zhcon.sourceforge.net/)，貌似能解决这个问题，前提是你的系统不能是x86_64的，得是i386的，你所在的用户组必须得在Video组，`usermode username -aG video`，切，还是算了吧，在X-window下的终端使用就很好了。

## w3m

[lynx](http://lynx.browser.org/)，老牌的文本浏览器，至今一直在维护

[w3m](http://w3m.sourceforge.net)，较新的文本浏览器，可以支持图片预览，更新速度不及lynx，但貌似很好用。


> http://jasonwryan.com/blog/2011/05/05/w3m/

> http://comments.gmane.org/gmane.linux.debian.user/405449

还有就是w3m-img可以在tty终端显示图片，很酷的功能，但目前所知的只有在tty终端并使用root用户（root用户有framebuffer）下才能显示，而X-window下的终端模拟器下，目前还不能显示，得继续查找方法。

以上结论，参见：

> http://pc-freak.net/blog/browse-the-web-graphically-in-text-console-ttys-with-w3m-img-and-links2-on-debian-ubuntu-fedora-and-centos-linux/

> http://askubuntu.com/questions/387678/how-to-enable-display-of-framebuffer-images-e-g-in-w3m-fbi-within-byobu-pts


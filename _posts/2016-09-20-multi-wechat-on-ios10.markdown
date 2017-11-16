---
layout: post
title: 在 iOS 9 或者 iOS 10 上微信多开
category: Document
tags: network
date: 2016-09-20 13:12:18
published: true
summary: 两个微信在 iOS 上多开
image: pirates.svg
comment: true
---

方法：使用 safari 浏览器打开网址 http://m.pc6.com/s/144125，

![示例图](http://images.weiphone.net/data/attachment/forum/201601/06/185630o7bhoogip7npp22p.png)

点击立即下载，弹出新界面，点击安装

安装后点击“微信分身版”会提示“不受信任的企业开发者”，只需进入设置->通用->描述文件->找到提示的那个添加信任就可以了。

这种方法原理很简单，就是修改 wechat 的描述信息，让 iOS 认为是两个应用就可以了。

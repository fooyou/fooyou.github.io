---
layout: post
title: 如何使用 phpstorm + xdebug 调试本地 php 工程
category: Document
tags: php xdebug
date: 2016-05-06 11:05:55
published: true
summary: php 小白日记
image: pirates.svg
comment: true
---

本人 PHP 小白，但喜欢 jetbrain 的 IDE，所以就直接拿 PhpStorm 来玩玩 PHP 了，在大部分时间使用 C，Python 和 Java 开发时，对 IDE 的需求不是太强烈，但遇到大～工程时，IDE 的调试优势就凸显出来了。原以为 jetbrain 会开发个一级棒的 PHP IDE，但 PHP 调试貌似很 bitch，看官方的 xdebug 可以调试，就试试吧，大爷的，为什么都 IDE 了，这些东西还得自己搞，搞毛！

## Pre: 安装 php

```
$ sudo apt-get install php5-dev
```

## Step 1: 安装 xdebug

安装 xdebug

官方文档里说，要在 php.ini 里添加

```
zend_extension="/usr/local/php/modules/xdebug.so"
```

谁知道 php.ini 在哪里，`locate php.ini` 一下，N 多地方都有这个文件，改哪个？

> https://xdebug.org/docs/install
> http://icephoenix.us/php/how-to-setup-local-php-debugging-with-phpstorm-and-xdebug/

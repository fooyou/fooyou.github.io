---
layout: post
title: linux上安装phpmyadmin并配置使用
category: Document
tags: linux
date: 2015-11-18 18:37:10
published: true
summary: 安装和配置linux PHP开发环境时，想安装phpmyadmin却不知道怎么都特么不好用……
image: pirates.svg
comment: true
latex: false
---

安装和配置linux PHP开发环境时，想安装phpmyadmin却不知道怎么都特么不好用……，后来终于在ubuntu论坛上找到了解决方法：

用apt-get安装完phpmyadmin后，要做以下两件事：

1. 执行命令：

```
$ sudo a2enconf phpmyadmin
$ sudo /etc/init.d/apache2 reload
```

> https://help.ubuntu.com/community/phpMyAdmin

---
layout: post
title: Ubuntu 下迁移 MySQL db 到其他目录
category: Document
tags: mysql linux
date: 2016-07-12 15:07:41
published: true
summary: 近期导入巨量数据，结果发现服务器硬盘不够用了，综合考虑了一下，决定先移动数据库到另一大分区，记录一下过程。
image: pirates.svg
comment: true
---

**The table is full**

MySQL 默认的 table 表的大小是 16M，修改 /etc/mysql/my.cnf 或者 (ubuntu 16.04) /etc/mysql/conf.d/my.cnf

参见：

> http://www.2cto.com/database/201501/373939.html

> http://www.jb51.net/article/62967.htm


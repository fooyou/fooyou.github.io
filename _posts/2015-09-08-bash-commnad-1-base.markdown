---
layout: post
title: bash命令基础篇
category: Document
tags: linux
date: 2015-09-08 09:12:40
published: true
summary: 本文介绍bash命令一些常用的基本命令
image: pirates.svg
comment: true
latex: false
---

## 文档帮助类命令

  命令  |             描述             | 例子
--------|------------------------------|-------------
 man    | 阅读命令文档                 | man bash
 apropos| 搜索手册页名称和描述         | apropos ssh
 help   | 查看bash内置命令，如jobs，fg | help bg

## 重定向

使用`>`和`<`来重定向输出和输入，明白`>>`和`<<`是在文件尾追加而不是重写文件，明白`|`可用来重定向管道。

## 文件管理

  命令  |             描述             | 例子
--------|------------------------------|-------------
 ls     | 列举当前目录下的内容         | ls -la
 more   | 文件细读过滤器               | more readme.md
 less   | 和more相反的文件阅读器       | less readme.md
 tail   | 查看文件的最后部分           | tail readme.md
 ln     | 在文件间建立链接             | ln -s target directory
 chown  | 改变文件所有者和组           | chown -hR root:joshua /u
 chmod  | 改变文件的读写可执行模式     | chmod 777 run.sh
 du     | 评估文件空间占用             | du 01.mp3
 df     | 文件系统磁盘空间使用情况     | df -a
 mount  | 挂载文件系统                 | mount /dev/foo /dir
 fdisk  | 操作磁盘分区表               | fdisk -l（root权限）
 mkfs   | 创建linux文件系统（新建磁盘）| -
 lsblk  | 列出块设备                   | lsblk
 
## 网络管理

  命令  |             描述             | 例子
--------|------------------------------|-------------
 ip     | 显示/操作路由，设备，隧道等  | ip address
 ifconfig| 配置网络接口                | ifconfig -a
 dig    | DNS 查找工具                 | dig @server name type

## Bash快捷键

 可使用`man readline`查看Bash所有快捷键，以下列出我常用的快捷键

  快捷键  |             描述
----------|------------------------------
 ctrl-w   | 删除键入的最后一个单词
 ctrl-u   | 删除键入整行
 alt-b    | 按单词向前移动光标
 alt-f    | 按单词向后移动光标
 ctrl-k   | 从光标出删除到行尾
 ctrl-l   | 伪清屏

## 常用功能及命令

   命令   |            功能描述
----------|------------------------------
 cd -     | 回到上一个工作路径
 find . -name '*.py' | xargs grep graphviz | 查找当前目录及子目录下所有的后缀为`.py`的并且其中含有`graphviz`的文件
 pstree -p| 展示进程树
 netstat -lntp / ss -plat | 检查哪些进程在监听端口（默认检查TCP端口，-u检查UDP端口）
 uptime / w | 查看系统已经运行的时间
 wc       | 查看文件行数，词数，字节数，etc



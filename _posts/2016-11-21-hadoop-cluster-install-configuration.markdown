---
layout: post
title: Hadoop-1.2.1 + HBase 完全分布式安装配置过程
category: Document
tags: bigdata
date: 2016-11-21 15:11:25
published: true
summary: 详细步骤一步一步搭建完全分布式的 Hadoop 
image: pirates.svg
comment: true 
---

*由于项目原因，需要搭建基于 Hadoop-1.2.1 的分布式存储系统，尽管安装很简单，有大量官方和博客上的文档可供参考，但为了知识总结还是一步一步记录下来，以供参考*

## 环境

本次搭建使用基于 Ubuntu 16.04 的 ElementaryOS 0.4 系统，使用 VirtualBox 先安装一个虚拟系统 hs_a，在安装 Hadoop 之前，需要做一些准备工作。

### 1. 创建 hadoop 用户

```sh
$ sudo useradd -m hadoop -s /bin/bash    # 创建可以登录的 Hadoop 用户，并使用 /bin/bash 做为 shell。
$ sudo passwd hadoop                     # 设置密码
```

### 2. 安装 SSH 并配置无密码登录

Cluster、单节点模式需要用到 SSH 登录，需要安装 SSH server：

```sh
$ sudo apt-get install openssh-server openssh-client
```

## 参考

> http://blog.csdn.net/lemon_tree12138/article/details/51607646
> http://blog.csdn.net/poisonchry/article/details/27205003
> http://www.powerxing.com/install-hadoop-cluster/

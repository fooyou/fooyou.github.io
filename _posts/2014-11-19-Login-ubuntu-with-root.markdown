---
layout: post
title: Ubuntu开启root账户登录的方法
category: Document
tags: ubuntu
year: 2014
month: 11
day: 19
published: true
summary: Ubuntu下root登录方法
image: pirates.svg
comment: true
---

Ubuntu 默认是不允许使用root登录的，要使用root权限，就必须使用sudo命令来执行，安全繁琐。

两个步骤打开：

**1:** 设置root密码，开启root账号：

```
sudo passwd root
```

**2:** 使用命令su 或者 sudo -s 登录启用

**3:** 修改lightdm配置，修改 /etc/lightdm/lightdm.conf.d/10-ubuntu.conf 文件

```
vi /etc/lightdm/lightdm.conf.d/10-ubuntu.conf
```

从后面添加如下配置，重启即可看到root账户登录项

```
greeter-show-manual-login=true
```
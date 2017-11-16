---
layout: post
title: linux + ssh + chrome 翻墙
category: Document
tags: linux
date: 2016-10-11 17:10:02
published: true
summary: 我的翻墙选择
image: pirates.svg
comment: true
---

Windows 上翻墙使用 freegate 或者 psiphone3 很方便。

Linux MacOS 使用 ssh 比较方便。下面介绍一下步骤：

## Step 1: 申请 ssh

提供免费 ssh 的网站比较多，比如：

- fastssh.com (免费)
- skyssh.com (免费)
- contassh.com (免费)
- freeshell.org (免费)
- Redhat cloud (免费)
- Amazon cloud 
- ...


总之搜一个，注册一个，比如 fastssh.com，注册一个 7 天的用户，系统会给你如下信息：

```
Your Username fastssh.com-qwerter has been successfully created !

Username SSH : fastssh.com-qwerter
Password SSH : 12345678
Host IP Addr : usa.serverip.co

Date Created : 11-October-2016
Date Expired : 18-October-2016

Account will expire on 18-October-2016.
You allowed use this account up to two (2) multi-bitvise, else you will be automatically disconnected from the server
```

另外 fastssh 提供了一个永久免费的账号，只是速度很慢。但我在写这篇日记时没有连接成功，账号如下：

```
Username: fastssh.com
Password: fastssh.com
Host IP Addr: free.fastssh.com
Openssh Port: 22 & 143
```

## Step 2: 使用 ssh 绑定 ssh server 到 本地 

```
$ ssh -fNC -D 7070 fastssh.com-qwerter@usa.serverip.co
fastssh.com-qwerter@usa.serverip.co's password: 
```

输入密码后就把 ssh server 绑定到了本地的 7070 端口。

-f -N -C -D  什么意思？ man ssh 自己查看，我曾试过 ssh -qfnNC 但是不成功，使用 -fNC 就能成功。

## Step 3: 启动 chromium-browser

```
$ chromium-browser --proxy-server="socks://localhost:7070"
```

若是在 Mac 上，则如何启动

```
$ open -a /Applications/Google\ Chrome.app --args --proxy-server="socks5://localhost:7070"
```


OK！使用 google 搜索吧（程序员离开 google 该怎么活？！）

当然，你也可以为 chrome 装个 Proxy SwitchShape 的插件，把这个选项配置在插件里(试过了，这个不好用)

好记性不如烂笔头，要做好笔记！


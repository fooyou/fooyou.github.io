---
layout: post
title: npm设置代理
category: Document
tags: linux
date: 2015-08-19 14:42:20
published: true
summary: npm设置代理
image: pirates.svg
comment: true
---

有许多方法设置npm代理，要让nodejs和npm绕过代理，我还是喜欢使用npm命令：

设置http代理

```
$ npm config set proxy http://proxy.xxx.com:8080/
```

但是访问的确实https的源，所以还得设置https-proxy:

```
$ npm config set https-proxy http://proxy.xxx.com:8080/
```

万事大吉！

 

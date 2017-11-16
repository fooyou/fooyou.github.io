---
layout: post
title: linux生成随机字符串
category: Document
tags: linux
date: 2015-10-21 13:20:10
published: true
summary: 可用"/dev/urandom"生成
image: pirates.svg
comment: true
latex: false
---

## 生成全字符字符串

```
$ cat /dev/urandom | strings -n C | head -n L
```

## 生成数字加字母的随机字符串

```
$ cat /dev/urandom | sed 's/[^a-zA-Z0-9]//g' | strings -n C | head -n L
```

其中：C代表字符数，L代表行数。

> http://blog.sina.com.cn/s/blog_54229ea6010096g4.html


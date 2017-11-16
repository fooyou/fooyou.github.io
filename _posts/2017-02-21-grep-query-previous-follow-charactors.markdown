---
layout: post
title: grep 命令查询命中字符串的前后 n 个字符
category: Document
tags: linux
date: 2017-02-21 11:02:20
published: true
summary: 
image: pirates.svg
comment: true
---

```
$ echo "some123_string_and_another" | grep -o -P '.{0,3}string.{0,4}'
23_string_and
```


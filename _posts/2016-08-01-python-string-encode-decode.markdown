---
layout: post
title: Python 字符串加密解密总结
category: Document
tags: python
date: 2016-08-01 10:08:24
published: true
summary: 敏感信心如何存储呢？
image: pirates.svg
comment: true
---

*今天是八一建军节，现在的我能无忧无虑的工作生活，幸福并感激着！*

敏感信息私密信息需要加密以确保相对安全，本文对个人使用过的方法做个总结。

字符串加密和解密一般来讲加解密繁琐程度和安全程度是成正比的，接下来就从简单到复杂介绍一下 Python 中的方法。


## Case 1：Base64

如果你只是想混淆一下字符串来愚弄一下粗心的观察者，那么 base64 是个不错的选择，不需要依赖第三方库，使用非常简单，但安全系数上来讲就像一层薄薄的纸。

```python
>>> import base64
>>> base_str = 'SouthChinaSea'
>>> encrypt_str = base64.encodestring(base_str.encode('utf-8'))
>>> encrypt_str
b'U291dGhDaGluYVNlYQ==\n'
>>> base64.decodestring(encrypt_str)
b'SouthChinaSea'
```

## Case 2: 维吉尼亚密码（Vigenere cipher）

多表查询密码

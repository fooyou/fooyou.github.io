---
layout: post
title: "ImportError: bad magic number in 'time': b'\\x03\\xf3\\r\\n'"
category: Document
tags: python
year: 2015
month: 05
day: 04
published: true
summary: Python import模块出现如是错误时，不要着急，把.pyc删掉就可以了
image: pirates.svg
comment: true
---

编写的Python脚本，用Python3一直好用，有时用Python2.7调用也好用，但今天竟然发生了这个错误：

```
    ImportError: bad magic number in 'time': b'\x03\xf3\r\n'
```

不知道这是神马错误。Google之，如下解决：

把运行脚本产生的.pyc删掉，再运行就OK了。

据说这是Python升级到3.4.2，切用之前版本运行过包含`import datatime`模块所导致的。

Anyway, the problem is solved!
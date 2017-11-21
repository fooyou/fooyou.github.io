---
layout: post
title: 合并多个Map至一个
category: Document
tags: python
year: 2015-05-04
published: true
summary: 
mathjax: false
highlight: true
---

## 问题

如何合成多个Map，以方便查询

## 解决方案

比如有以下字典：

```python
    a = {'x': 1, 'z': 3}
    b = {'y': 2, 'z': 4}
```

现在你想在这两个字典中查询（先查询a如果没查到再查询b）。一个简单的做法就是使用`collections`里的`ChainMap`类，如下：

```python
    from collections import ChainMap
    c = ChainMap(a, b)
    print(c['x'])
    print(c['y'])
    print(c['z'])
```

## 讨论

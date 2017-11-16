---
layout: post
title: Python3.4 使用 virtualenv
category: Document
tags: python virtualenv
year: 2015
month: 06
day: 15
published: true
summary: 总之，折腾了一圈
image: pirates.svg
comment: true
---

感觉 virtualenv 虚拟 python 环境很好用，很多同行前辈都推荐。所以刚入python的我也用了，python 2.7 装完了，用着挺酷，然后开始装 python3.4 的，装完了发现很多问题，后来又发现 python3.4 内带了这个功能，当时就一脑门黑线。真心的应该用pycharm 看一眼python3.4 都内置了神马东东。好吧，使用内置的最后终于可行了，但仍有一些待解决的问题，以后再说吧。


__最后 python3.4 是这么好用的:__

```
$ mkdir myproject
$ cd myproject
$ pyvenv venv
$ source ./venv/bin/activate
$ 
$ deactivate
```

__问题：__

1. 不知道 `pyvenv` 和使用 `virtualenv`有什么差别，至少生成的文件是不一样的。
2. `virtualenv-3.4` 总是不成功，不知道为什么。
3. `python3 -m venv xxx` 提示错误返回，不知道为什么。

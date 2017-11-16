---
layout: post
title: pip 使用国内源安装 python 模块
category: Document
tags: python
date: 2016-12-22 11:12:40
published: true
summary: 公司的国际线路慢的要死，使用 pip 安装模块一直断断...
image: pirates.svg
comment: true 
---

国内的 pypi 的镜像应该有好多，比如豆瓣、淘宝、清华...

清华的 index-url: https://pypi.tuna.tsinghua.edu.cn/simple

使用:

```sh
$ pip install -i https://pypi.tuna.tsinghua.edu.cn/simple tensorflow
```

修改 pip 默认的的配置，可新建 ~/.pip/pip.conf，并添加如下内容：

```
[global]

index-url=https://pypi.tuna.tsinghua.edu.cn/simple
```

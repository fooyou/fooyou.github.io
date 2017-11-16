---
layout: post
title: pip如何使用cache
category: Document
tags: pip python
year: 2015
month: 07
day: 09
published: true
summary: maven默认状态下是把所有包都cache下来的，重复的包就不用多次下载，但是发现使用virtualenv下的pip不是，同样一个包不同环境会多次下载，幸亏看到了这个方法：
image: pirates.svg
comment: true
---

maven默认状态下是把所有包都cache下来的，重复的包就不用多次下载，但是发现使用virtualenv下的pip不是，同样一个包不同环境会多次下载，极大的浪费了时间和带宽。

在[pip新闻](https://web.archive.org/web/20081207054341/http://pip.openplans.org/news.html)里找到如下配置方法：

> Added support for an environmental variable $PIP_DOWNLOAD_CACHE which will cache package downloads, so future installations won’t require large downloads. Network access is still required, but just some downloads will be avoided when using this.

所以，在`~/.bash_profile`里添加这个变量：

linux下：

```
export PIP_DOWNLOAD_CACHE=$HOME/.pip_packages
```

MacOS下：

```
export PIP_DOWNLOAD_CACHE=$HOME/Library/Caches/pip-packages
```

这样，你的pip会像maven一样工作了。

> PS: pip7.0默认就有Cache了，不需要配置了。

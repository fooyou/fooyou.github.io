---
layout: post
title: linux常用的查找命令
category: Document
tags: linux 
date: 2015-11-10 11:03:10
published: true
summary: linux 常用的查找命令汇总
image: pirates.svg
comment: true
latex: false
---

```
$ find /tmp -name test -type f -print | xargs /bin/rm -f
```

查找并删除 /tmp 目录下的所有名为 test 的文件。

**注意**：不适用于文件名中含有换行单引或双引号或者空格的文件。

```
$ find /tmp -name test -type f -print0 | xargs -0 /bin/rm -f
```

查找并删除 /tmp 目录下的所有名为 test 的文件。

适用于查找路径中含有特殊字符文件名的查找并删除。


```
$ find . -type f -exec file '{}' \;
```

对当前目录和其下的子目录中的文件执行 `file` 命令 `'{}' \;` 用于保护文件

---
layout: post
title: svn 故障说目录 lock 需要执行 Cleanup，可 Cleanup 的时候又说 lock
category: Document
tags: network
date: 2016-10-10 18:09:03
published: true
summary: 很久很久没用 svn 了，我是 git 粉，今天用 svn 竟然遇到这个错误
image: pirates.svg
comment: true
---

## 问题描述：

在用 svn checkout 或 update  代码时，因为某种原因（比如磁盘空间不足，比如网络链接不稳定），svn 会把当前目录 lock 住，不让进行下一步操作并提示需要 Cleanup，当你执行 Cleanup 后又会提醒你有 lock 无法 Cleanup， 死循环不工作。

## 解决方法：

使用 SQLiteSpy 打开 svn 根木下下的 .svn/wc.db，然后 Tables -> WC\_LOCK，双击WC\_LOCK 查看里面有无东西（一般这时肯定有），然后别客气删掉：

```
delete from WC_LOCK
```

F9 执行，然后再更新就好用了！

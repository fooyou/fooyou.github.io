---
layout: post
title: linux的history命令
category: Document
tags: linux
date: 2015-10-27 14:13:10
published: true
summary: 许多命令是重复的不断用的，虽然熟能生巧，但总打这些命令的确很烦人，所以可以通过history命令来简化重复命令的使用。
image: pirates.svg
comment: true
latex: false
---

history命令可以快速的查找已使用的命令，更妙的是可以通过日期和时间戳去查找相应时间段使用过的命令。

1. 列出所有执行过的命令

```
$ history
```

列出如下命令：

```
 1001  ps -e|grep w3m
 1002  echo $JAVA_HOME
 1003  cd Downloads/
```

2. 列出所有执行过命令的日期和时间戳

```
$ export HISTTIMEFORMAT='%F %T  '
$ history
```

export的那个变量将打印出日期和时间戳

```
 1022  2015-11-02 10:11:00  rm .fuse_hidden000*
 1023  2015-11-02 10:11:00  rm .fuse_hidden000000*
 1024  2015-11-02 10:11:00  rm .fuse_hidden0000006f00000005 
 1025  2015-11-02 10:11:00  rm .fuse_hidden0000006f00000007 
```

3. 过滤历史消息

```
$ export HISTIGNORE='ls -l:pwd:date:'
$ history
```

4. 忽略重复命令

```
$ export HISTCONTROL=ignoredups
$ history
```

5. 取消export设置

```
$ unset export HISTCONTROL
$ unset export HISTTIMEFORMAT
```

6. 永久设置export变量

可以在`~/.bashrc`或者`~/.bash_profile`中设置这几个参数

7. 特定用户的history保存在什么地方

保存在`~/.bash_history`文件下

8. 禁止history保留历史

公司安全考虑禁止history记录可以在`~/.bashrc`或者`~/.bash_profile`中设置变量`HISTSIZE=0`。

而如果只是想在一个会话里禁止记录历史，可以在终端里`export HISTSIZE=0`

9. 删除或者清楚history记录

```
$ history -c
```

10. 搜索历史记录

```
$ history | grep cmd
```

11. 搜索最后执行的相关命令

按下`Ctrl + r`，输入简单搜索字符，终端会搜索出相匹配的最近执行的命令，按`enter`键执行搜出的命令，`esc`键返回。

```
(reverse-i-search)`pa': mvn package
```

12. 重调最后执行的相关命令

`history`列出历史记录及其序号，然后`!n`，就可以执行第n号命令

```
$ !1200
```

将执行第1200号命令


> http://www.tecmint.com/history-command-examples/



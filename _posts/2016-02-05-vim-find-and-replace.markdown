---
layout: post
title: vim中查找和替换的使用方法
category: Document
tags: vim
date: 2016-02-05 11:02:28
published: true
summary: 工欲善其事必先利其器，vi/vim 中可以使用：s 命令来替换字符串。该命令有很多种不同细节使用方法，可以实现复杂的功能。
image: pirates.svg
comment: true
---

*vi/vim 中可以使用：s 命令来替换字符串。该命令有很多种不同细节使用方法，可以实现复杂的功能，记录几种在此，方便以后查询。*

         命令        |             描述
---------------------|----------------------------------
：s/vivian/sky/      | 替换当前行第一个 vivian 为 sky
：s/vivian/sky/g     | 替换当前行所有 vivian 为 sky
：n，$s/vivian/sky/  | 替换第 n 行开始到最后一行中每一行的第一个 vivian 为sky
：n，$s/vivian/sky/g | 替换第 n 行开始到最后一行中每一行所有 vivian 为 sky。n 为数字，若 n 为 .，表示从当前行开始到最后一行
：%s/vivian/sky/     | （等同于：g/vivian/s//sky/）替换每一行的第一个 vivian 为 sky
：%s/vivian/sky/g    | （等同于：g/vivian/s//sky/g）替换每一行中所有 vivian 为 sky。可以使用 # 作为分隔符，此时中间出现的 / 不会作为分隔符
：s#vivian/#sky/#    | 替换当前行第一个 vivian/ 为 sky/
：%s+/oradata/apras/+/user01/apras1+ | （使用+ 来替换 / ） /oradata/apras/替换成/user01/apras1/

### 替换确认

vim也有替换确认，需要在后面加上命令 `c` （confirm），例如：

替换当前所有 vivian 为 sky：

```
: s/vivian/sky/g
```

加上 `c` 后就会有确认提醒：

```
: s/vivian/sky/gc
```

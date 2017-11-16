---
layout: post
title: linux查看各个账户开关机情况
category: Document
tags: linux
date: 2016-02-17 19:02:15
published: true
summary: 突然想知道我的 elementary os 今天的开关机情况，如何办，使用 last 命令
image: pirates.svg
comment: true
---

## last 命令

```
$ last
```

显示如下：

```
joshua   tty1                          Wed Feb 17 19:21   still logged in
joshua   pts/1        :0               Wed Feb 17 19:15 - 19:20  (00:05)
joshua   :0           :0               Wed Feb 17 18:56   still logged in
reboot   system boot  3.16.0-50-generi Wed Feb 17 18:55 - 19:24  (00:29)
joshua   tty1                          Wed Feb 17 15:29 - down   (02:22)
joshua   :0           :0               Wed Feb 17 15:26 - down   (02:25)
reboot   system boot  3.16.0-50-generi Wed Feb 17 15:24 - 17:51  (02:27)
joshua   pts/2        :0               Sat Feb  6 09:08 - 10:49  (01:41)
joshua   :0           :0               Sat Feb  6 09:03 - down   (01:46)
reboot   system boot  3.16.0-50-generi Sat Feb  6 09:03 - 10:50  (01:46)
joshua   pts/1        :0               Fri Feb  5 18:15 - down   (00:11)
joshua   pts/1        :0               Fri Feb  5 18:04 - 18:15  (00:10)
joshua   tty1                          Fri Feb  5 10:00 - 17:24  (07:24)
joshua   :0           :0               Fri Feb  5 09:55 - down   (08:31)
reboot   system boot  3.16.0-50-generi Fri Feb  5 09:55 - 18:27  (08:31)
joshua   pts/11       :0               Thu Feb  4 15:44 - 15:47  (00:03)
joshua   tty1                          Thu Feb  4 10:27 - down   (08:29)
joshua   pts/1        :0               Thu Feb  4 10:23 - 10:23  (00:00)
joshua   :0           :0               Thu Feb  4 10:14 - down   (08:43)
reboot   system boot  3.16.0-50-generi Thu Feb  4 10:13 - 18:57  (08:44)
joshua   tty1                          Wed Feb  3 10:10 - down   (08:44)
joshua   :0           :0               Wed Feb  3 10:06 - down   (08:49)
reboot   system boot  3.16.0-50-generi Wed Feb  3 10:06 - 18:55  (08:49)
joshua   tty1                          Tue Feb  2 10:06 - down   (09:57)
joshua   :0           :0               Tue Feb  2 10:05 - down   (09:57)
reboot   system boot  3.16.0-50-generi Tue Feb  2 10:05 - 20:03  (09:58)
joshua   pts/8        :0               Mon Feb  1 17:24 - 18:57  (01:32)
joshua   pts/8        :0               Mon Feb  1 17:16 - 17:19  (00:02)
joshua   pts/10       :0               Mon Feb  1 16:54 - 16:59  (00:05)
joshua   tty1                          Mon Feb  1 13:34 - down   (05:24)
joshua   pts/5        :0               Mon Feb  1 11:14 - 13:28  (02:14)
joshua   pts/4        :0               Mon Feb  1 11:13 - 13:28  (02:15)
joshua   pts/2        :0               Mon Feb  1 11:12 - 13:28  (02:16)

wtmp begins Mon Feb  1 11:12:30 2016
```

Pretty Cool!


---
layout: post
title: linux 修改硬件时间和系统时间
category: Document
tags: linux
date: 2016-07-22 09:07:09
published: true
summary: 如何修改 linux 系统时间
image: pirates.svg
comment: true
---

## linux 系统时钟介绍

linux系统时钟有两个，一个是硬件时钟，即BIOS时间，就是我们进行CMOS设置时看到的时间，另一个是系统时钟，是linux系统Kernel时间。当Linux启动时，系统Kernel会去读取硬件时钟的设置，然后系统时钟就会独立于硬件运作。有时我们会发现系统时钟和硬件时钟不一致，因此需要执行时间同步，下面就分享一下时间设置及时钟同步的命令使用方法。

## date 命令

date 命令可以修改系统时间，使用方法：

```
$ date
```

说明： 查看当前系统时间

```
$ date -s 07/22/16
```

说明：将日期设定为 2016年7月22号

```
$ date -s 09:58:21
```

说明：将日期设定为 09:58:21

```
$ date 0722165809.21
```

说明：将日期设定为2016年7月22号09:58:21

## hwclock 命令

hwclock 可修改硬件时钟，并可同步到系统时钟。

使用方法：

```
$ hwclock --show
```

说明：查看当前硬件时间

```
$ hwclock --set --date="07/22/16 09:58:21"
```

说明：设定格式（月/日/年 时：分：秒）

```
$ hwclock --hctosys
```

说明：同步系统时钟


## 修改时区

tcselect 命令可修改当前用户的时区

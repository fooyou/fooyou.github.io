---
layout: post
title: elementary os 蓝牙无法自动连接
category: Document
tags: linux elementary bluetooth
date: 2023-02-02 02:02:20
published: true
summary: 
image: pirates.svg
comment: true
---

### 问题：

使用 elementary os 一段时间后，发现蓝牙键盘无法连接来，每次都得删掉设备后，重新配对连接。


### 解决方法

使用 linux rfkill 命令打开设备，命令如下：

```bash
$ rfkill unblock bluetooth
$ systemctl enable bluetooth.service
$ systemctl start bluetooth.service
```

### 更多

rfkill 命令是 linux 用来管理系统中无线设备的命令，可以打开 unblock 和 关闭 block 被 block 住的设备。

rfkill list: 可以查看无线设备是否被 Soft blocked 或 Hard blocked。

```bash
$ rfkill list
1: phy0: Wireless LAN
        Soft blocked: no
        Hard blocked: no
3: hci0: Bluetooth
        Soft blocked: yes
        Hard blocked: no
```

如果发现有无线设备被 blocked，那么可以用 `rfkill unblock [device name]` 来打开

```bash
$ rfkill unblock bluetooth
```

或者:

```bash
$ rfkill unblock 3
```

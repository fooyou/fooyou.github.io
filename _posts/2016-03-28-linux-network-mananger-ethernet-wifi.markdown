---
layout: post
title: NetWorkManager 管理 ethernet 和 WiFi
category: Document
tags: linux network
date: 2016-03-28 17:03:37
published: true
summary: ubuntu 创建了多个 ip 链接，不知道怎么切换，于是 google 出了这篇文章，很实用。
image: pirates.svg
comment: true
---

**Network**

ubuntu 使用 NetworkManager 管理以太网和 WiFi

查看所有可见连接：

```
$ nmcli con list
```

## Ethernet

断开以太网（重启后不再自动连接）:

```
$ nmcli dev disconnect iface eth0
```

ip 配置存放在 `/etc/NetworkManager/system-connections/` 下，文件名即是以太网网络连接的 id 名。这里可以配置多个 ip，但只能启用一个。

```
$ ls /etc/NetworkManager/system-connections/
Home Work
$ cat /etc/NetworkManager/system-connections/Home
[802-3-ethernet]
duplex=full

[connection]
id=Home
uuid=a8e8c1be-36bb-4094-b187-219f2f9eaf09
type=802-3-ethernet

[ipv6]
method=auto

[ipv4]
method=auto
```

启用 Home:

```
$ nmcli con up id "Home"
```

关闭 Home：

```
$ nmcli con down id "Home"
```

## 静态 IP

修改静态 IP 地址，可以编辑上面的 Home 文件，比如修改 [ipv4]：

```
$ vim /etc/NetworkManager/system-connections/Home
```

```
[ipv4]
method=manual
address1=192.168.1.100/24,192.168.1.1
dns=8.8.8.8;8.8.4.4;
```

address1 的格式如下：

```
    address1=<IP>/<prefix>,<route>
```

## WiFi

列出可见的 WiFi 列表：

```
$ nmcli dev wifi
```

创建一个新的名为“Mycafe”的连接，然后使用密码“caffeine”连接至 SSID 名为“Cafe Hotspot 1”：

```
$ nmcli dev wifi connect "Cafe Hotspot 1" password "caffeine" name "Mycafe"
```

断开 “Mycafe” 连接：

```
$ nmcli con down id "Mycafe"
```

启用 “Mycafe” 连接：

```
$ nmcli con up id "Mycafe"
```

显示 Wifi on/off 状态：

```
$ nmcli nm wifi
```

打开 wifi：

```
$ nmcli nm wifi on
```

关闭 wifi：

```
$ nmcli nm wifi off
```

参考：

> http://wiki.t-firefly.com/index.php/Firefly-RK3288/UbuntuServerNote/en

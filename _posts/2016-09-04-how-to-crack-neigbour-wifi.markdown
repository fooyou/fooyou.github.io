---
layout: post
title: 如何破解邻居家的 WiFi 密码
category: Document
tags: network
date: 2016-09-04 20:09:03
published: true
summary: 有时急需上网，但是自家的网络不好用，那么如何不道德的盗用邻居家的 WiFi 呢？
image: pirates.svg
comment: true
---

现在的 WiFi 密码一般都采用 WPA/WPA2 进行加密，它使用非常强壮的密码存储机制可以显著的减慢自动破解程序的速度，使用 PBKDF2 解密函数来解密通过 4096 次迭代的加密哈希算法加密过的密码，在2012年测试只需几分钟即可破解，但若用它来破解 WPA/WPA2 加密的 WiFi 密码则需要几天或者几周或者几月。

另外最低要求 8 个字符且还使用了 SSID 进行了混淆（防止骇客使用密码字典进行有效的破译）。

但这并不是说，WiFi 密码不能轻松的破解。首先来了解一下终端设备和无线路由之间的四次握手过程：

![WiFi 连接握手过程](http://cdn.arstechnica.net/wp-content/uploads/2012/08/4way.png)


具体思路：

1. 抓取握手数据 [deauth frame](https://wilder.hq.sk/OpenWeekend-2005/foil04.html)；
2. 利用强大的云计算暴力破解；


实施方案：

1. 使用 [Silica wireless hacking tool](https://www.immunityinc.com/products-silica.shtml) (是要钱的) 抓取 deauth frame
2. 把握手数据文件上传至 [CloudCracker](https://www.cloudcracker.com/) 网站上，利用它的云计算能力破解出密码(也是要收费的)

参考：

> http://arstechnica.com/security/2012/08/wireless-password-easily-cracked/

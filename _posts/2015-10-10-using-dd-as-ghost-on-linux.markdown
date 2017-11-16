---
layout: post
title: linux下如何像ghost一样备份系统
category: Document
tags: linux
date: 2015-10-10 14:29:55
published: true
summary: 使用过Windows Ghost的小朋友都想在linux下也有这么个工具，其实开放的linux下有多种实现方案，本文介绍一种使用dd命令的方法。
image: pirates.svg
comment: true
latex: false
---

Windows下Ghost可以克隆整个系统的镜像，然后在新的电脑上恢复，相当简单。Ghost安装系统比使用安装镜像安装要快的多，因为Ghost磁盘存储是连续的，且安装过程中不需要回答任何问题。

为什么不能使用Ghost来备份和恢复linux系统呢？因为Ghost只能识别很少老旧的lunux文件系统，其无法识别grub和LILO等引导加载程序。

其实linux下有个著名的g4l—ghostForlinux，但貌似无法实现对一个或者几个分区的恢复。想搞深点怎么办呢？

Ghost原理是什么？不就是数据复制吗？linux下dd命令不就是最强大的数据复制工具吗。既然如此，为什么还要使用g4l这么复杂的工具呢？一条dd命令就可以帮助实现任意复杂的镜像复制和恢复的需求了，不管是grub，还是ext4，btrfs，FAT32，NTFS…dd都可以搞定。

## fdisk和dd命令

打开终端，执行如下命令：

```
$ sudo fdisk -u -l
```

将看到类似如下信息：

```
Disk /dev/sda: 500.1 GB, 500107862016 bytes
255 heads, 63 sectors/track, 60801 cylinders, total 976773168 sectors
Units = 扇区 of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x2bd2d38b

   设备 启动      起点          终点     块数   Id  系统
/dev/sda1   *        2048      718847      358400    7  HPFS/NTFS/exFAT
/dev/sda2          718848   210817023   105049088    7  HPFS/NTFS/exFAT
/dev/sda3       210817024   871911423   330547200    7  HPFS/NTFS/exFAT
/dev/sda4       871913470   976771071    52428801    5  扩展
/dev/sda5       972652544   976771071     2059264   82  Linux 交换 / Solaris
/dev/sda6       871913472   972652543    50369536   83  Linux

Partition table entries are not in disk order
```

此命令可以查看所有磁盘上的所有分区的尺寸和布局情况。-u，让start和end中的数字的单位是512字节，也就是一个sector扇区的大小。

可见上述机器的linux系统装在了/dev/sda6分区中，下面将演示使用dd命令备份该分区的步骤：

**步骤一：**

制作linux live cd的启动U盘。

**步骤二：**

U盘启动，进入linux live系统，然后查看硬盘分区情况，找到需要备份的分区/dev/sda6

**步骤三：**

执行dd命令：

```
$ dd bs=512 count=[fdisk命令中最大的end数+1] if=/dev/sda6 of=/sysbackup.img
```

这样就把我们需要的分区数据全部copy到sysbackup.img文件中了，镜像制作完成！！

然后，我们把U盘插到其他系统上，用U盘启动，进入linux live CD，打开终端，执行如下命令：

```
$ dd if=sysbackup.img of=/dev/sda
```

完成后，拔掉U盘，启动计算机，就可以看到我们的linux系统已经安装完毕了！

**_注意：_**不要直接在计算机上用本地磁盘启动系统后执行dd命令生成本地磁盘镜像，而应该用liveCD启动计算机。因为计算机在运行时会对系统盘产生大量写操作。直接对运行中的系统生成的镜像，在恢复到其他硬盘上时，很可能会无法启动！！

## What's More!!

这种生成镜像和恢复系统的方法同样适用于非linux操作系统，因为linux的fdisk命令能够识别任意系统下的分区格式，fdisk并不关心分区上的文件系统，甚至有无文件系统都不关心。fdisk总是可以报告分区占用了哪些扇区。

dd命令同样不关心磁盘的文件系统格式，它只是简单地按照要求从指定的位置复制多少数据而已。

dd命令实现镜像备份和恢复，比Ghost软件简单和强大多了。使用Ghost软件，依然需要用户进行复杂而危险的磁盘分区操作。

而使用fdisk和dd命令，一切如此简单。

## 和压缩命令一起使用

dd备份的分区大的话产生的镜像就会很大。存储传输镜像不太方便，可以和gzip压缩命令一起使用：

```
$ dd bs=512 count=[fdisk命令中最大的end数+1] if=/dev/sda6 | gzip -6 > /sysbackup.img.gz
```

还原时命令：

```
$ gzip -dc /sysbackup.img.gz | dd of=/dev/sda
```

**提醒：**

如果你把镜像恢复到另一台计算机上，你可能会发现你的网卡是eth1，而不是eth0。这是因为/etc/udev/rules.d/70-persistent-net.rules文件把你做镜像的计算机的网卡作为eth0登记了。

如果你的网络脚本对eth0进行了处理，而没有对eth1进行处理，那么不修改网络脚本，你可能就无法上网了。

也许你会希望在做镜像之前，先删除 /etc/udev/rules.d/70-persistent-net.rules 文件。这样你恢复镜像时，网卡的名字就是eth0。   就不会造成你在恢复后的计算机上无法上网的问题了。

**注意：**

最好在dd生成镜像之前，先umount所有 if和of 设备的分区。这样可以确保在dd的过程中文件系统没有被改变。

在完成dd（生成镜像和恢复镜像）后，执行sudo sync，确保数据被真正写入到硬盘上。

另外，如果你想要对整个硬盘进行备份和恢复，而不是只备份和恢复部分分区，那么就请把dd命令中的 count=[fdisk命令中最大的end数+1]   去掉。bs=512也可以去掉。

> http://blog.jobbole.com/90978/

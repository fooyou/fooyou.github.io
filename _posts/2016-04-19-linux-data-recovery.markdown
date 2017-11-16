---
layout: post
title: linux 下文件和磁盘恢复
category: Document
tags: datarecovery linux
date: 2016-04-19 10:04:45
published: true
summary: 误删了文件怎么办？误删了磁盘怎么办？电脑重装了，但突然想起磁盘上还有重要的文件，怎么办？
image: pirates.svg
comment: true
---

## 恢复分区

**parted 命令**

Q：分区被弄丢了，分区表里不显示了？怎么办？

A：使用 `parted` 命令。parted 命令是 ubuntu 内置的命令。

不过使用时要注意，得使用 liveCD  进入系统后使用，因为恢复分区需要更改分区表，而更改分区表则要保证所有的分区都在非挂载状态。

当 liveCD 启动后，使用 `swapoff -a` 命令取消所有设备页面和交换功能，然后就可以使用 `parted` 恢复有问题的磁盘了，假如 /dev/sda 有问题，则：

```
$ parted /dev/sda
```

然后使用 rescue 选项：

```
$ rescue START END
```

START 是分区开始的偏移值， END 同理，如果 parted 命令找到潜在的分区，它将询问你是否加入分区表。


**testdisk**

恢复分区的另一个工具名叫 testdisk ，运行它将扫描整个媒体设备并提供菜单驱动的方式来恢复分区。

```
$ sudo testdisk
```


**gpart**

GPart 使用“猜测”方式扫描并重建分区表，以下命令将以默认方式扫描第一磁盘：

```
$ sudo gpart /dev/sda
```

你可以通过以下命令恢复“猜测”出来的分区表，注意，在使用前你必须确认“猜测”来的分区表是否正确，否则后果很严重。

```
$ sudo gpart -W /dev/sda /dev/sda
```

## 镜像损坏的设备，文件系统或者驱动

恢复损坏的文件之前，可以使用这两个工具制作镜像：

- **GNU ddrescue**：包的名字叫 gddrescue，但命令却是 ddrescue。
- **dd_rescue**：包的名字叫 ddrescue，命令确实 dd_rescue，是又老又慢的需要和其他脚本共同使用才能达到 ddrescue 效果。

比如你的 PC 的硬盘分区 `/dev/sda3` 坏掉了，你可以把它镜像到 USB 磁盘中（如果/dev/sda3 大于4G，那么 USB 磁盘格式不能为 MSDOS（VFAT），请使用 ext3 格式化 U 盘），救援命令如下：

```
$ ddrescue -r 3 /dev/sda3 /media/usbdrive/image /media/usbdrive/logfile
```

-r: 意思是如果遇到读取错误，则重试3次。

或者连续读取：

```
$ ddrescue -r 3 -C /dev/sda3 /media/usbdrive/image /media/usbdrive/logfile
```

一般情况下，可以这么使用：

1. 拷贝尽可能多的数据，不进行重试和存储扇面分割：

    ```
    $ ddrescue --no-split /dev/sda3 imagefile logfile
    ```

2. 然后使用非缓存读，重试先前出错的地方3次：

    ```
    $ ddrescue --direct --max-retires=3 /dev/sda3 imagefile logfile
    ```

3. 如果还是失败，那么使用 direct 和 retrim 参数重新读取所有的扇区：

    ```
    $ ddrescue --direct --retrim --max-retires=3 /dev/sda3 imagefile logfile
    ```

其他例子：


**例子一：把 ext2 格式的分区 `/dev/hda2`  救援到 `/dev/hdb2`**

    ```
    $ ddrescue -r3 /dev/hda2 /dev/hdb2 logfile
    $ e2fsck -v -f /dev/hdb2
    $ mount -t ext2 -o ro /dev/hdb2 /mnt
    ```

**例子二：救援 CD-ROM（`/dev/cdrom`）**

    ```
    $ ddrescue -b 2048 /dev/cdrom cdimage logfile
    ```

    然后，就可以把 cdimage 写入到 CD-ROM 中去了。

**做镜像时空间不足了？**

使用 gnu ddrescue 的 logfile，你可以把镜像做到多个驱动盘中，然后再合并起来，以下是合并碎片的命令行：

```
$ sudo losetup /dev/loop1 /media/Drive1/image
$ sudo losetup /dev/loop2 /media/Drive2/image
$ sudo mdadm -B /dev/md0 -l linear -n 2 /dev/loop1 /dev/loop2
```

合并后的镜像在 /dev/md0，然后：

```
$ sudo mdadm -S /dev/md0
$ sudo losetup -d /dev/loop1
$ sudo losetup -d /dev/loop2
```

使用 ddrescue 需要注意的是，如果恢复时频繁发生 I/O Error 可能会把 /var/log/kern.log 和 /var/log/syslog 填充的非常大，为了避免这种事情发生，最好把这些文件 link 到 /dev/null


到现在为止，我们已经制作了恢复镜像，那么接下来：


## 从救援镜像中恢复文件系统

即便不能从救援镜像中恢复文件系统，也可以视图从中恢复文件。如果你镜像了整个驱动，你可以把镜像中独立的分区使用`offset`选项挂载到正在运行的文件系统中。SleuthKit 命令可以帮忙。

```
$ sudo apt-get install sleuthkit
```

mmls 命令可以帮助查看镜像中的分区情况：

```
$ mmls file -b
DOS Partition Table
Offset Sector: 0
Units are in 512-byte sectors
     Slot    Start        End          Length       Size    Description
00:  -----   0000000000   0000000000   0000000001   0512B   Primary Table (#0)
01:  -----   0000000001   0000000031   0000000031   0015K   Unallocated
02:  00:01   0000000032   0001646591   0001646560   0803M   DOS FAT16 (0x06)
03:  00:00   0001646592   0002013183   0000366592   0179M   DOS FAT16 (0x06)

```

假如我们要挂载偏移量为 32 的 DOS 分区，乘以 512 计算字节数：

```
$ bc
bc 1.06
Copyright 1991-1994, 1997, 1998, 2000 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'.
32 * 512
16384
quit
```

挂载分区：

```
$ sudo mount -o loop,offset=16384 file mnt
```

若要挂载经典的 NTFS 格式的分区，使用：

```
$ sudo mount -t ntfs -o r,force,loop,offset=32256 file mnt
```

**从救援镜像中恢复独立的文件**

*Foremost*

Foremost 可以众多的文件系统中恢复文件，包括 fat, ext3, NTFS，可以从 live cd 中安装和运行。它可以从镜像中也可以从驱动中直接恢复文件。如果驱动有硬件问题，使用 gnu ddrescue 制作救援镜像先。

假设在 hda 上你丢失了文件，那么首先从其他磁盘或分区中创建一个可写的文件夹用来恢复文件(假设有一个大存储的 sdb)：

```
$ sudo mount /dev/sdb1 /recovery
$ sudo mkdir /recovery/foremost
```

然后运行 foremost

```
$ sudo foremost -i /dev/hda -o /recovery/foremost
```

如果是恢复救援镜像：

```
$ sudo foremost -i image -o /recovery/foremost
```

恢复的文件将被 root 所拥有，所以改变它们的关系来使用它们：

```
$ sudo chown -R yourname:yourname /recovery/foremost
```

使用 -w 来切换到恢复文件只有一个 audit ：

```
$ sudo foremost -w -i /dev/hda -o /recovery/foremost
```

只恢复一种类型的文件，可使用 -t 指定类型：

```
$ sudo foremost -t jpg -i /dev/hda /recovery/foremost
```

可见的文件类型如下：

- jpg
- gif
- bmp
- avi
- exe
- mpg
- wav
- riff
- wmv
- mov
- pdf
- ole
- doc
- zip
- rar
- htm
- cpp
- all


**Scalpel**

Scalpel 和 foremost 类似，但有一些改进。它从镜像文件或者 raw device 中直接把符合头尾定义的文件解压出来，默认的文件头尾定义存放在 /etc/scalpel/scalpel.conf 中，如果想指定恢复的文件，你得修改这个文件。

```
$ sudo scalpel FILE -o Directory
```

**Magic Rescue**

Magic Rescue 使用“magic bytes”来识别文件类型和数据，可通过“recipes”来扩展更多的文件类型。

Note：大多数的文件类型需要安装其他软件，请阅读 /usr/share/magicrescue/recipes。

例子：加入你刚从 /dev/sda1 上误删了一个美女的写真集，有 gzip 压缩包和 png 文件，那么你可以这样恢复：

```
$ mkdir ~/recover
$ sudo magicrescue -r gzip -r png -d ~/recover /dev/sda1
```

PS: 貌似很好用，但怎么从一个分区的某个目录中恢复文件呢？这样恢复需要太大的空间了。


**Photorec**

Photorec 最初被设计成从数字照相机存储卡中恢复图片文件的，但现在已经扩展成能恢复80多种文件了，它是 testdisk 的一部分，你可以安装 testdisk 获得它。

在镜像文件上运行 photorec：

```
$ sudo photorec imagefilename
```

从设备上直接恢复文件，直接运行 photorec，会获得一个可见设备的菜单

```
$ sudo photorec
```

更多请参考[这里](http://www.psychocats.net/ubuntucat/recoverdeletedfiles/)

**recoverjpeg**

恢复 jpeg 文件的

**Ntfsprogs**

恢复 NTFS 格式的


参考：

> https://wiki.archlinux.org/index.php/File_recovery
> https://help.ubuntu.com/community/DataRecovery

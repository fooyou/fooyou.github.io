---
layout: post
title: VirtualBox 使用拾零
category: Document
tags: virtualbox
date: 2016-09-27 11:39:25
published: true
summary: VirtualBox 在 Linux 平台下的使用拾零，包括 USB 的使用和 VDI 磁盘扩容等。
image: pirates.svg
comment: true
---

*工作需要从心爱的 elementary os 又迁到了 Win 7 系统，不甘心的我因为开发机比较强悍，所以果断安装了 eos，然后用 VirtualBox 虚拟 Win 7 开发，老长时间不用了一些小技巧又忘了，笔记一下吧*

## 1. Linux 下使用 VirtualBox 安装虚拟系统后，如何打开 USB 功能

需要两个步骤：

1. 安装 [VirtualBox 扩展包](http://download.virtualbox.org/virtualbox/5.1.6/Oracle_VM_VirtualBox_Extension_Pack-5.1.6-110634.vbox-extpack)以便打开 USB 2.0 和 3.0（注意扩展包的版本号和 Vbox 的要一致）

2. Linux 端需要把当前用户添加到 vboxusers 用户组里，否则 VBox 没有访问 USB 口的权限：

	```
	$ sudo usermod -G vboxusers -a joshua
	```
	把 joshua 替换为当前用户。

	可由 `cat /etc/group | grep vboxusers` 查看结果

重启即可使用


## 2. VDI 虚拟磁盘扩展容量

也需要两个步骤，借助 VirtualBox 提供的工具 vboxmanage 扩容

MACOS 的以下命令在 /Applications/VirtualBox.app/Contents/MacOS/ 下，可以 cd 到这个目录，然后使用 vb 命令。

1. 查看虚拟磁盘命令：

	```
	$ vboxmanage list hdds
	UUID:           a2502312-474c-4684-a59f-5ac624f87127
	Parent UUID:    base
	State:          locked write
	Type:           normal (base)
	Location:       /media/joshua/My_Document/VirtualOS/Win7.vdi
	Storage format: VDI
	Capacity:       81920 MBytes
	Encryption:     disabled
	```

	得到 VDI 的 UUID 来进行扩容时的索引

2. 扩容命令：

	```
	$ vboxmanage modifyhd a2502312-474c-4684-a59f-5ac624f87127 --resize 102400(M)
	```

	扩容至 100G，打开 VirtualBox 可以发现该虚拟系统的磁盘已变为 100G

3. 进入虚拟机进行扩容：

	Windows 系统（Win 7+）可以 计算机->管理->磁盘->选中系统盘，扩展卷。
	
	Linux 系统:

        1. 需要用到 gparted（sudo apt-get install gparted）
        2. 打开它 sudo gparted
        3. 记住 linux-swap 分区，一般为 2G
        4. 右键 linux-swap，点击 `Swapoff`
        5. 右键 linux-swap，点击 `Delete`
        6. 点击绿色对号，应用当前设置
        7. 右键点击 /dev/sda2 磁盘，然后删除它 delete
        8. 右键点击 /dev/sda1，点击 Resize，扩展的时候预留出 linux-swap 的空间（2G）
        9. 右键点击余下的（2G）空间创建扩展分区
        10. 选择 `linux-swap`
        11. 点绿色对号，应用修改1
        12. 在 `linux-swap` 分区上，右键然后点击 `swapon`
        13. 右键 linux-swap 分区，点击 Information，拷贝 UUID 或记住 /dev/sdax 信息
        14. sudo vim /etc/fstab 修改之前的交换分区，为新的交换分区
        15. reboot -> OK!

## 3. 修改 VDI 虚拟磁盘路径

修改 VDI 磁盘位置后，虚拟机不能工作，除了新建虚拟机还可修修改虚拟机的配置文件，比如 linux 下其默认配置文件在 ~/VirtualBox MS，比如我的 Win 7 的配置文件名叫 Win7.vbox：

```
└── Win7
    ├── Logs
    │   ├── VBox.log
    ├── Win7.vbox
    └── Win7.vbox-prev
```

打开 Win7.vbox 把其下的所有路径修改为新路径就可以了。

## 4. 修改 VDI 虚拟磁盘的 UUID

有时候需要复制多个 VDI 磁盘用于测试，由于复制后有相同的 UUID 值，导致 VBox 无法加载新的磁盘，可以用如下命令修改 UUID：

```sh
$ VBoxManage internalcommands sethduuid "path/to/vdi"
```


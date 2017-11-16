---
layout: post
title: 查看硬件信息的 linux 命令
category: Document
tags: linux
date: 2016-05-20 10:05:24
published: true
summary: 
image: pirates.svg
comment: true
---

本文搜集了 linux 上快速查看硬件信息或是配置信息的命令。


    命令     |    说明
-------------|--------------------------------
 lscpu       | 显示 CPU 架构信息
 lshw        | 显示所有可见的硬件详细信息，比如 CPU，内存，硬盘等等
 hwinfo      | 比 lshw 还要详细，但需要额外安装（Ubuntu上）
 lspci       | 查看 PCI
 lsscsi      | 查看 SCSI
 lsusb       | 查看 USB
 lsblk       | 查看 block device
 df          | 硬盘分区使用情况
 fdisk       | 是用来修改分区的，但也可以查看分区信息
 mount       | 挂载/卸载文件系统
 free        | 查看内容剩余情况
 /proc 文件  | /proc 目录下包含硬件的虚拟文件信息。 cat /proc/cpuinfo
 

---
layout: post
title: 查看 Linux 进程内存占用情况
category: Document
tags: linux
date: 2017-03-03 12:03:39
published: true
summary: 查看 Linux 进程内存的命令
image: pirates.svg
comment: true
---

top、ps、pmap

## (一): ps 查看进程内存等信息

```
$ ps -e -o 'pid,comm,args,pcpu,rsz,vsz,stime,user,uid'
```

rsz 是实际内存，vsz 是虚拟内存，可以使用管线 grep 过滤，sort -nrk5 等排序

## (二): top 命令

```
　　PID：进程的ID
　　USER：进程所有者
　　PR：进程的优先级别，越小越优先被执行
　　NInice：值
　　VIRT：进程占用的虚拟内存
　　RES：进程占用的物理内存
　　SHR：进程使用的共享内存
　　S：进程的状态。S表示休眠，R表示正在运行，Z表示僵死状态，N表示该进程优先值为负数
　　%CPU：进程占用CPU的使用率
　　%MEM：进程使用的物理内存和总内存的百分比
　　TIME+：该进程启动后占用的总的CPU时间，即占用CPU使用时间的累加值。
　　COMMAND：进程启动命令名称
```

## (三): pmap 查看详情

```
$ pmap -d 6129
```


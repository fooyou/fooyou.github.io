---
layout: post
title: Ubuntu上编译ffmpeg的动态链接库（.so）
category: Workaround
tags: ffmpeg ubuntu build
year: 2015
month: 04
day: 08
published: true
summary: ubutnu中使用动态链接库的场景 
image: pirates.svg
comment: true
---

##问题

*从[ffmpeg官网](https://ffmpeg.org/)上下载了源码，然后按照其[编译文档](https://trac.ffmpeg.org/wiki/CompilationGuide/Ubuntu)在ubuntu(12.04~14.04)上进行编译，然后发现生成的链接库里没有动态库.so文件。如何生成动态链接库呢？*

##解决步骤

1. configure时选项要使用 `--enable-shared` 来打开动态链接库选项。但是这样编译出来的文件里可能没有动态库。有可能是第2步的原因导致。
2. configure时，prefix默认为/usr/local，然而该目录默认情况下是不在ldconfig下的，所以要添加一下方法如下：

    ```
    sudo vi /etc/ld.so.conf.d/libc.conf
    ```

    然后添加如下路径就可以了（PS：14.04上在创建该文件时就自动写好了）

    ```
    /usr/local/lib
    ```

    然后应用该配置，生成详细信息：

    ```
    sudo ldconfig -v
    ```

3. 在源码目录下，就可以build了：

    ```
    ./configure --enable-shared && make && make install
    ```

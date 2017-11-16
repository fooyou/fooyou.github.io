---
layout: post
title: ffmpeg 使用手册
category: Document
tags: ffmpeg
date: 2017-06-08 14:06:33
published: true
summary: 超赞，超爱 ffmpeg 有木有？
image: pirates.svg
comment: true
---

ffmpeg 视频处理的终极神器，你值得学习！

## 语法

```
$ ffmpeg [[infile options][-i infile]] ... {[outfile options] outfile} ...
```

## 描述

options 修饰随后的指定文件，所以参数顺序很重要，同一个 option 可以在一行命令中多次出现，每个 option 作用于随后的输入或输出文件。

**设定输出文件的视频比特率(即码率)**

```
$ ffmpeg -i input.mp4 -b 64k output.mp4
```

比特率(bitrate)是什么？关系到画质和音质。

*比特率是指每秒传送的比特(bit)数。单位为 bps(Bit Per Second)，比特率越高，传送的数据越大。声音中的比特率是指将数字声音由模拟格式转化成数字格式的采样率，采样率越高，还原后的音质就越好。 视频中的比特率（码率）原理与声音中的相同，都是指由模拟信号转换为数字信号的采样率。比特就是二进制里面最少的单位，要么是0，要么是1。比特率与音频压缩的关系简单的说就是比特率越高音质就越好，但编码后的文件就越大；如果比特率越少则情况刚好翻转*


**设定输出文件的帧率**

```
$ ffmpeg -i input.mp4 -r 24 output.mp4
```

帧率(frame rate)是什么？关系到流畅度。

*帧率是指每秒钟传输的图片的帧数，也可以理解为图形处理器每秒钟能够刷新几次，通常用fps（Frames Per Second）表示。每一帧都是静止的图象，快速连续地显示帧便形成了运动的假象。高的帧率可以得到更流畅、更逼真的动画。每秒钟帧数 (fps) 愈多，所显示的动作就会愈流畅。一般来说30fps是可以接受的，所以要避免动作不流畅的最低fps是30。除了30fps外，有些计算机视频格式，例如AVI，每秒只能提供15帧。我们之所以能够利用摄像头来看到连续不断的影像，是因为影像传感器不断摄取画面并传输到屏幕上来，当传输速度达到一定的水平时，人眼就无法辨别画面之间的时间间隙，所以大家可以看到连续动态的画面。*

**强制输入文件的帧率为 1，输出文件的帧率为 24**

```
$ ffmpeg -r 1 -i input.m2v -r 24 output.mp4
```

可能要求输入文件为原始格式的文件。

默认情况下, ffmpeg 的转换为无缺损的，如果没有指定音视频的一些参数(比如编码格式、帧率、码率等)，ffmpeg 转换的时候会使用原文件的数据。


## 选项

### 普通选项

普通选项适用于 ff* 工具。

#### -L

显示许可

#### -h, -?, -help, --help

显示帮助

#### -version

显示版本

#### -formats

显示可见格式 D: 解码，E: 编码。

#### -codecs

显示可见编解码

```
Codecs:
 D..... = Decoding supported
 .E.... = Encoding supported
 ..V... = Video codec
 ..A... = Audio codec
 ..S... = Subtitle codec
 ...I.. = Intra frame-only codec
 ....L. = Lossy compression
 .....S = Lossless compression 
```

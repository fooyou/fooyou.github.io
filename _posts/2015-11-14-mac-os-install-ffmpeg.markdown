---
layout: post
title: Mac OS下安装ffmpeg和ffplay
category: Document
tags: macos
date: 2015-11-14 11:55:10
published: true
summary: 其实就是介绍使用brew安装ffmpeg的，那还用介绍吗？还是有必要的。
image: pirates.svg
comment: true
latex: false
---

不知道ffmpeg什么东西，她是一个开源项目，可以将任何格式的音视频文件进行编解码转换播放等等等等，基本可以这么说你对音视频相关的任何需求她都能办到，但首先你得会用她。开源领域内神一样的VLC，还有Media Player Classic都是在用ffmpeg的codec库。

Mac OS有着很美的操作界面，人性化的设计让许多开发者爱不释手，更主要的是其基于FreeBCD的内核架构对Unix和linux友好，所以现在越发收开发者喜爱。当然中国有很多使用Windows系统的开发者，曾经我也是不知道linux等开源系统意味着什么，但当我加入到linux行列才发现，我擦，这才是我的世界啊。唠叨完毕，下面进入正题，如何安装ffplay在MacOS上。

为什么要使用ffplay，ffplay是ffmpeg为测试其codec lib而开发的基于命令行的播放器，用户交互当然没法跟其他播放器比，但对于开发者来说，这家伙神奇至极，今天从magnet上下载了高清蚁人，mkv封包的，但用其他播放器播放效果不好，不是声音不同步就是不能播放，所以干脆在家也用ffplay播放吧。

在Mac上安装linux社区的软件常用有两种方式：一、使用MacPorts；二、使用Homebrew，因为我用的是brew所以只说用它安装的方式。

## 一、安装Homebrew

1. 使用Ruby安装Homebrew：

    ```
    $ ruby -e "$(curl -fsSL https://raw.github.com/Homebrew/homebrew/go/install)"
    ```

2. 使用brew安装ffmpeg：

    ```
    $ brew install ffmpeg --with-ffplay
    ```

除了安装选项 `--with-ffplay`外还有更多的选项如下：

```
–with-fdk-aac  (Enable the Fraunhofer FDK AAC library)
–with-ffplay  (Enable FFplay media player)
–with-freetype  (Build with freetype support)
–with-frei0r  (Build with frei0r support)
–with-libass  (Enable ASS/SSA subtitle format)
–with-libcaca  (Build with libcaca support)
–with-libvo-aacenc  (Enable VisualOn AAC encoder)
–with-libvorbis  (Build with libvorbis support)
–with-libvpx  (Build with libvpx support)
–with-opencore-amr  (Build with opencore-amr support)
–with-openjpeg  (Enable JPEG 2000 image format)
–with-openssl  (Enable SSL support)
–with-opus  (Build with opus support)
–with-rtmpdump  (Enable RTMP protocol)
–with-schroedinger  (Enable Dirac video format)
–with-speex  (Build with speex support)
–with-theora  (Build with theora support)
–with-tools  (Enable additional FFmpeg tools)
–without-faac  (Build without faac support)
–without-lame  (Disable MP3 encoder)
–without-x264  (Disable H.264 encoder)
–without-xvid  (Disable Xvid MPEG-4 video encoder)
–devel  (install development version 2.1.1)
–HEAD  (install HEAD version)
```

OK，开始播放吧！

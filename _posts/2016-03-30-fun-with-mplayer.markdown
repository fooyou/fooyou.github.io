---
layout: post
title: mplayer 在 tty 终端中播放视频
category: Document
tags: linux mplayer tty
date: 2016-03-30 16:03:17
published: true
summary: tty 中 mplayer ASCII 播放真心没法看，好在 mplayer 支持 framebuffer 播放。
image: pirates.svg
comment: true
---

## 以 ASCII 形式播放视频

需要 libaa 或者 libcaca

```
$ mplayer -vo aa SampleVideo.mp4
```
`-vo`: video output driver
aa: ASCII 字符

输出彩色字符：

```
$ mplayer -vo caca SampleVideo.mp4
```

## 以 framebuffer 播放

```
$ mplayer -vo fbdev2 SampleVideo.mp4
```

可能出现错误，因为每个 framebuffer 都有指定的分辨率大小，如果看高清的视频需要修改 fb 的大小，或者缩放以进行播放：

```
$ mplayer -vo fbdev2 -zoom -xy 640 SampleVideo.mp4
```

参数什么意思，请`man mplayer` 查看。

不得不说，mplayer 是个神器，amazing。

参考：

> https://bbs.archlinux.org/viewtopic.php?id=36832

---
layout: post
title: 使用 mplayer 在墙纸上播放视频
category: Document
tags: mplayer linux
date: 2016-04-07 16:04:25
published: true
summary: 没有毛用，但是很 Cool
image: pirates.svg
comment: true
---

Ubuntu 上：

```
# 不让 GNOME 处理 Desktop
gconftool-2 -type bool -set /apps/nautilus/preferences/show_desktop false
# 在根窗口上全屏播放视频
mplayer -rootwin -fs sample.mkv
```

可惜我没成功，我用的是 elementary os，使用 gconf-editor 找了个相似的配置项，修改了 gnome 下的 desktop -> background，然并卵。

参考：

> http://blogs.harvard.edu/hoanga/2008/02/06/how-to-play-mplayer-in-the-desktop-fullscreen-on-ubuntu/
> http://lifehacker.com/299410/use-a-screensaver-as-desktop-wallpaper

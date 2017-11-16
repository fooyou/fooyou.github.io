---
layout: post
title: linux下抓取屏幕至动态gif图像中
category: Document
tags: linux screencapture
date: 2015-09-07 16:42:27
published: true
summary: GeekStyle linux下抓取屏幕至动态gif图像中，用到的命令包ffcast(ffmpeg), convert(imagmagick)
image: pirates.svg
comment: true
latex: false
---

### 以下是实际效果：

![sample](https://raw.githubusercontent.com/fooyou/fooyou.github.io/master/img/posts/2015-09-07_17-34-51.gif)

操作步骤：

1. 启动脚本 ./screen2gif
2. 选中要录制的屏幕区域
3. 返回脚本，按q退出录制
4. 等待一会儿，gif生成完毕。
5. OK，到Home/Pictures下查看吧！

## 怎么实现：

### 屏幕抓取

FFcast可以使用户选择屏幕一块区域并使用第三方工具比如ffmpeg进行屏幕录像。

ffcast是Arch linux社区的一些黑客的荣耀作品，可以在github上找到[ffcast](https://github.com/lolilolicon/FFcast2)，它只依赖于`bash`和`ffmpeg`，并使用[xrectsel](https://github.com/lolilolicon/xrectsel)进行矩形选择。

还可以对ffmpeg命令进行扩展。我使用`-r 15`来进行每秒种捕获15帧，`-codec:v huffyuv`进行无损录制。


### Gif化

Imagemagick可以读取`.avi`视频并有一些gif优化的小把戏能减少gif文件的大小同时还能保持好的画面质量。`convert`命令的`-layers Optimize`参数可启动Gif优化，我使用的最后的convert命令如下：

```sh
$ convert -set delay 1x40 -layers Optimize $TMP_AVI $HOME/Pictures/$(date +%Y-%m-%d_%H:%M:%S).gif
```

其中 -delay 1x40 ：指每秒40帧

### 最后的脚本代码

将以下代码另存为：screen2gif，并添加可执行权限，即可使用了。

```sh
TMP_AVI=$(mktemp /tmp/outXXXXXXXXXX.avi)
ffcast -s % ffmpeg -y -f x11grab -show_region 1 -framerate 15 \
    -video_size %s -i %D+%c -codec:v huffyuv                  \
    -vf crop="iw-mod(iw\\,2):ih-mod(ih\\,2)" $TMP_AVI         \
    && convert -set delay 1x40 -layers Optimize $TMP_AVI $HOME/Pictures/$(date +%Y-%m-%d_%H:%M:%S).gif  \
    && rm $TMP_AVI
```

_注：_

ffmpeg和ffcast的安装略。

参考：

> http://unix.stackexchange.com/questions/113695/gif-screencasting-the-unix-way

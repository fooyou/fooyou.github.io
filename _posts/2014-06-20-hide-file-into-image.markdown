---
layout: post
title: 将文件隐藏在图片中的方法（linux/windows）
category: Document
tags: image
date: 2014-06-20
published: true
summary: 为什么要把文件隐藏在图片里？潜入
image: pirates.svg
comment: true
---

## linux

------

如下步骤隐藏：

- 将图片1.jpg和file文件单独存放到一文件夹下，比如/tmp/temp
- ctrl + alt + t调出terminal：

    ```bash
    cd /tmp/temp
    ```
- 执行如下命令：

    ```bash
    cat 1.jpg file >> img.jpg
    ```

如此找回文件：

```bash
    unzip img.jpg
```

## Windows

------

如下步骤隐藏：

- 将图片1.jpg和file文件单独存放到一文件夹下，比如c:\temp
- Win + R -> 输入cmd -> cd c:\temp
- 执行如下命令：

    ```
    copy /b img.jpg + 1.jpg file
    ```
    *_注意：_图片一定要在file文件前面*

如此找回文件：

使用7-zip等压缩软件打开img.jpg即可。
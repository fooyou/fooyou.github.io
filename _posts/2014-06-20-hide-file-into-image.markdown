---
layout: post
title: 将文件隐藏在图片中的方法（linux/windows）
category: Document
tags: image
year: 2014
month: 06
day: 20
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

    ```
    cd /tmp/temp
    ```
- 执行如下命令：

    ```
    cat 1.jpg file >> img.jpg
    ```

如此找回文件：

```
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
    *__注意：__图片一定要在file文件前面*

如此找回文件：

使用7-zip等压缩软件打开img.jpg即可。
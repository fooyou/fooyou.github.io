---
layout: post
title: 在Ubuntu或Debian上安装OpenCV
category: Document
tags: opencv linux
year: 2015
month: 06
day: 04
published: true
summary: 安装过程记录一下
image: pirates.svg
comment: true
---

**注意:** 本文档记录的内容只在Ubuntu 14.04 LTS上测试过, OpenCV版本为3.0.0.

安装OpenCV的过程很长，但是相对简单，可以源安装或者手动安装，因为要使用最新版且使用python，所以我选择的是手动安装。


## 从软件源安装OpenCV

```
    $ sudo apt-get install libopencv-dev python-opencv
```

这样安装通常不能安装最新版本，而是最稳定的版本。


## 手动安装OpenCV

安装最新的OpenCV前，要确保卸载已存在的包，使用命令`sudo apt-get autoremove libopencv-dev python-opencv`，然后按如下步骤：

### 1.确保Ubuntu或者Debian已更新

打开终端，执行如下命令：

```
    $ sudo apt-get update && sudo apt-get upgrade && sudo apt-get dist-upgrade && sudo apt-get autoremove
```

### 2.安装依赖包

**Build工具：**

```
    $ sudo apt-get install build-essential cmake
```

**GUI：**

```
    $ sudo apt-get install qt5-default libvtk6-dev
```

**媒体 I/O：**

```
    $ sudo apt-get install zlib1g-dev libjpeg-dev libwebp-dev libpng-dev libtiff5-dev libjasper-dev libopenexr-dev libgdal-dev
```

**视频 I/O：**

```
    $ sudo apt-get install libdc1394-22-dev libavcodec-dev libavformat-dev libswscale-dev libtheora-dev libvorbis-dev libxvidcore-dev libx264-dev yasm libopencore-amrnb-dev libopencore-amrwb-dev libv4l-dev libxine2-dev
```

**并行和线性代数库：**

```
    $ sudo apt-get install libtbb-dev libeigen3-dev
```

**Python：**

```
    $ sudo apt-get install python-dev python-tk python-numpy python3-dev python3-tk python3-numpy
```

**Java：**

```
    $ sudo apt-get install ant default-jdk
```

**文档：**

```
    $ sudo apt-get install doxygen sphinx-common texlive-latex-extra
```

### 3.下载并解压源码：

[下载](https://github.com/Itseez/opencv/archive/3.0.0.zip)

### 4.编译和安装OpenCV：

在源码根目录下，创建build文件夹，如下步骤：

```
    $ mkdir build
```

```
    $ cd build
```

```
    $ cmake -DWITH_QT=ON -DWITH_OPENGL=ON -DWITH_VTK=ON -DWITH_TBB=ON -DWITH_GDAL=ON -DWITH_XINE=ON -DBUILD_EXAMPLES=ON ..
```

```
    $ make -j4
```

```
    $ sudo make install
```

恭喜！安装完成。

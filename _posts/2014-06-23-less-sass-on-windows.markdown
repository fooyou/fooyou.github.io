---
layout: post
title: Windows下使用LESS和SASS的集成环境
category: Document
tags: less sass windows
year: 2014
month: 06
day: 23
published: true
summary: 为什么要less和sass？互联网风吹
image: pirates.svg
comment: true
---


## 一：分步安装SASS环境

**安装SASS和compass的方法：**

1. 安装Ruby，Windows下使用rubyinstaller-2.0.0-p353-x64.exe

2. 安装sass

    ```
    gem install sass
    ```

    如果命令行不好用，就手动安装：

    ```
    gem install --local <path/sass-3.2.12.gem>
    ```
3. 安装compass：
    a. 安装fssm：gem install --local <path/fssm-0.2.10.gem>
    b. 安装chunky_png：gem install --local <path/chunky_png-1.2.9.gem>
    c. 安装compass：gem install --local <path/ compass-0.12.2.gem>


## 二：集成环境

1. LESS：使用Crunch App，Crunch运行在Adobe air平台上，所以要先安装Adobe AIR。Crunch可以把分离的LESS文件合成一个CSS文件供Web APP使用，好处多多。

    a. 下载并安装Air：http://get.adobe.com/cn/air/

    b. 下载Crunch.air，并安装：http://crunchapp.net/

2. SASS：直接安装Prepros，所有的都集成在了里面，安装后直接使用，创建compass工程，使用sprite把icon全压缩在一张图上，并生成相应CSS文件。

    a. 下载并安装：http://alphapixels.com/prepros 第一步所有的东西都在这个App里了，东西很简单，看一个Demo就会。


## 参考资料

> http://compass-style.org/

> http://sass-lang.com/

> http://www.ruanyifeng.com/blog/2012/11/compass.html

> http://phpconf.hinablue.me/

> http://sass.hinablue.me/#slide1

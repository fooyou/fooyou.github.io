---
layout: post
title: 什么是颜色空间以及各颜色空间转换
category: Document
tags: color
date: 2016-10-25 14:10:48
published: true
summary: 制图打印时发生误差，原来是 RGB 到 CMYK 转换时使用了不一样的 ICC 文件......
image: pirates.svg
comment: true
---

## 颜色空间（Color Space）

人类为自然界中的可见光的色彩进行数学建模，这样才能在捕捉，显示以及印刷设备之间传递这些信息，由此，不同的颜色模型所表达不同色域的色彩，就有了颜色空间（Color Space），常见的颜色模型有 sRGB、AdobeRGB、ProPhotoRGB、CMYK等，这些模型都以可见光为基础，但各自色彩范围不同。

![Color Space 范围图](https://upload.wikimedia.org/wikipedia/commons/3/37/Colorspace.png)

*图 1： 颜色空间范围示意图*

可见，不同颜色空间之间可能存在一些彼此不包容的颜色。这些颜色在某个色彩空间环境中可以被显示出来，但在另一个颜色空间里可能无法再现出来。

### sRGB

sRGB 是美国的惠普与微软于1997年制定的标准色彩空间，在市场中占有较高份额。其是目前普通设备仪器中应用最广泛的色彩空间。同时也是范围最窄的色彩空间，它能显示的色彩非常有限，在专业领域中较少使用。

### AdobeRGB

AdobeRGB 是美国 Adobe 公司于 1998 年推出的色彩空间标准。其拥有宽广的色彩空间和良好的色彩层次表现，在专业摄影领域广泛应用。大多数数码相机都支持。与 sRGB 相比，其完全覆盖了 CMYK 色彩空间，因此 AdobeRGB 在印刷领域也得到广泛应用。

### CMYK

CMYK 是印刷行业以青（Cyan）、品红（Megenta）、黄（yellow）、黑（key-black）为原色的色彩空间模型。其构成了油墨印刷中的 CMYK 色彩空间。

由于 RGB 与 CMYK 不完全重合，因此可能导致一些在印刷品中出现的颜色无法在显示器中出现；而一些出现在显示器中的颜色，无法被印刷出来的问题。


*还有更多的色彩空间*

除了这些还有其他的一些色彩空间，都是用在不同领域的。

上述色彩空间又细化为很多标准，比如有 sRGB 有美国标准有日本标准有欧洲标准，还分为不同版本，而描述这些模型的文件叫做 ICC 或者是 ICM 文件，在做颜色转化时要指定待转换的颜色值及其对应的色彩空间标准（即 ICC 文件），还要指定要转换的色彩空间标准。这样才能正确转换。


## 开源代码库

### SampleICC

color.org 官网上有 SampleICC 的源码，可用于颜色空间的转换，[源码](http://sourceforge.net/projects/sampleicc/)，不过貌似只支持 Windows。

### Little CMS

google 了半天发现 Little CMS 是现在用的非常多的 color 转换库，开源跨平台，源代码结构清晰，易读，所以就选它了。


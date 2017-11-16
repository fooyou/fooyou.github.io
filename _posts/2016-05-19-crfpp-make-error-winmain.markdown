---
layout: post
title: crf++ linux 上编译出现 winmain.h 找不到的错误
category: Document
tags: crfpp
date: 2016-05-19 15:05:40
published: true
summary: 条件随机场（Conditional random field），给定一组输入随机变量的条件下可推断出另一组输出随机变量的条件概率分布模型，其特点是假设输出随机变量构成马尔科夫随机场。CRF 可用于不同的预测问题。
image: pirates.svg
comment: true
---

[crf++](https://github.com/taku910/crfpp) 是比较流行的 CRFs 开源的简单可定制化的 lib，专门为 NLP 一般性的需求设计，可用于命名实体识别、信息提取、和文本挖掘，等等。

其是用 C++ 开发的，但又封装了 python、java、ruby、swig 等模块，简单易用。

但是在 linux 上编译遇到了问题，错误如下：

```
g++ -DHAVE_CONFIG_H -I. -O3 -Wall -c -o crf_learn.o crf_learn.cpp
g++ -DHAVE_CONFIG_H -I. -O3 -Wall -c -o crf_test.o crf_test.cpp
crf_learn.cpp:9:21: fatal error: winmain.h: No such file or directory
#include "winmain.h"
```

很明显，这是在考虑 Windows 平台时添加的代码，而 linux 上找不到这个文件，可用如下方法解决：


```
$ ./configure
$ sed -i '/#include "winmain.h"/d' crf_test.cpp
$ sed -i '/#include "winmain.h"/d' crf_learn.cpp
$ make
$ sudo make install
```

一切顺利！

参见：

> https://github.com/taku910/crfpp/issues/18

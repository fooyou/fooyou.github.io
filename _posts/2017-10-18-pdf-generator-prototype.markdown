---
layout: post
title: pdf 生成模块投标方案
category: Document
tags: linux
date: 2017-10-19 10:10:10
published: true
summary: 使用 C 编写 pdf 生成模块的投标方案
image: pirates.svg
comment: true
---

## 我们的优势

- 资深 linux C 程序员 3 名（开发经验 6+ 年）；
- 使用 C++ 开发过类似模块；
    - 功能包括创建，解析和管理 pdf 文件，效率非常不错；
    - 页面内容包括文本、图片、pdf 嵌入、ttf, otf 字体；
    - 图片格式支持：jpg、tiff、png；
    - 文本支持 Unicode（包括特殊语种）


## 方案

我们的方案就是把我们以前的 Base 根据需求分离出相关模块并改成 C 版


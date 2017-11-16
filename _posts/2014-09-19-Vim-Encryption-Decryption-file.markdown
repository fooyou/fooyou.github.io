---
layout: post
title: vim/vi加密解密文件
category: Document
tags: vim
year: 2014
month: 09
day: 19
published: true
summary: Vim简单加解密文件
image: pirates.svg
comment: true
---


## 加密

使用vi打开文本，命令行模式输入X（大写 的）⇒ 回车，会提示输入密码，输入两次密码后，保存退出 ⇒ 完成

下次使用vim打开时输入密码，正常显示和编辑

## 解密

1. vi打开加密文本，输入正确密码，然后输入下面命令：

    ```
    :set key=
    ```
    回车保存即可解密

2. vi打开加密文本，输入正确密码，命令行输入:X，什么也不输入，然后保存。
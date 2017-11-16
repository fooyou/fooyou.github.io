---
layout: post
title: Eclipse通过proxy无法联网的原因
category: Workaround
tags: eclipse proxy ubuntu
year: 2014
month: 12
day: 11
published: true
summary: Ubuntu下，Eclipse通过代理怎么也连不上网络，如何解决？ 
image: pirates.svg
comment: true
---

google一番之后，原来这是Eclispe的bug，好在找到了一个回避方法，就是不要对SOCKS设置proxy，如下：

1. 打开*Network Connection Settings*
2. 选择*Active Provider**为*Manual*
3. 设置*HTTP/HTTPS*代理
4. 清除*SOCKS*代理（选中SOCKS并点击*Clear*按钮）
5. 重启Eclipse

Eclipse可以更新了

*本文参考：http://stackoverflow.com/questions/17338212/eclipse-kepler-not-connecting-to-internet-via-proxy*
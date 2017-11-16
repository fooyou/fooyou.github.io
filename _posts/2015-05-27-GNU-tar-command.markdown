---
layout: post
title: linux压缩/解压缩命令
category: Document
tags: linux compress
year: 2015
month: 05
day: 27
published: true
summary: 介绍linux下压缩命令的使用。
image: pirates.svg
comment: true
---

*linux 常用解压命令：*

压缩包 | 解压命令
-------|----------
*.tar  | tar -xvf *.tar
*.tar.gz | tar -xzvf *.tar.gz
*.tar.bz2 | tar -xjvf *.tar.bz2
*.tar.Z  | tar -xZvf *.tar.Z
*.gz | gunzip *.gz / gzip -d *.gz
*.bz2 | bzip2 -d *.bz2 / bunzip2 *.bz2
*.Z   | uncompress *.Z
*.rar | unrar e *.rar
*.zip | unzip *.zip

## tar命令

主操作模式：

命令 | 全称 | 描述
---|---------------|-------------
-A | --catenate, --concatenate |  追加 tar 文件至归档

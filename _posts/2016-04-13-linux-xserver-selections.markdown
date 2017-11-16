---
layout: post
title: tty 终端和和系统剪贴板交互
category: Document
tags: linux
date: 2016-04-13 11:04:09
published: true
summary: xclip 和 xsel 可以和系统剪贴板交互，但一直在使用 tmux，所以得把 tmux 的 Ctrl+b+[ 空格，选择后 Enter 这套流程 Copy 到系统剪贴板中。
image: pirates.svg
comment: true
---

为了能把 tmux 的复制的内容复制到 X11 系统剪贴板中，当前我是这么做的：

- 按完 C-b（Ctrl+b）后，按 [ 进入复制模式；
- 按下 Space 键进入文本选择选择；
- 选择文本后，按 Enter 键复制；
- 复制 tmux 缓存 到 X 剪贴板

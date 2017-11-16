---
layout: post
title: bash 中文文档
category: Document
tags: linux
date: 2015-09-08 10:55:01
published: true
summary: bash中文文档
image: pirates.svg
comment: true
latex: false
---

BASH(1)                                                                     BASH(1)

__名称(NAME)__

  bash - GNU Bourne-Again SHell（GNU命令解释程序“Bourne”二世）

__概述(SYNOPSIS)__

  bash [option] [command_string | file]

__版权(COPYRIGHT)__

  Bash is Copyright (C) 1989-2013 by the Free Software Foundation, Inc.

__描述(DESCRIPTION)__

  Bash 是一个与 sh 兼容的命令解释程序，可以执行从标准输入或者文件中读取的命令。 Bash 也整合了 Korn 和 C Shell (ksh 和 csh) 中的优秀特性。

  Bash 的目标是成为遵循 IEEE POSIX Shell and Tools specification (IEEE Working Group 1003.2，可移植操作系统规约： shell 和工具) 的实现。

__选项(OPTIONS)__

  除了在 set 内建命令的文档中讲述的单字符选项 (option) 之外，bash 在启动时还解释下列选项。

- -c string:  如果有 -c 选项，那么命令将从 string 中读取。如果 string 后面有参数 (argument)，它们将用于给位置参数 (positional parameter，以 $0 起始) 赋值。
- -i: 如果有-i选项，shell将交互地执行（interactive）.
- -l: 使得bash以类似登录shell(login shell)的方式启动（参见下面的启动(INVOCATION)章节） 。
- -r: 如果有 -r 选项，shell 成为受限的。
- -s: 如果有 -s 选项，或者如果选项处理完以后，没有参数剩余，那么 命令将从标准输入读取。这个选项允许在启动一个交互 shell 时可以设置位置参数。
- -D: 向标准输出打印一个以 $ 为前导的，以双引号引用的字符串列表。这在当前语言环境不是 C 或 POSIX 时，脚本中需要翻译的字符串。这个选项隐含了 -n 选项；不会执行命令。
- [-+]O [shopt_option]
    - shopt_option 是一个 shopt 内建命令可接受的选项 (参见下面的 shell内建命令(SHELL BUILTIN COMMANDS) 章节)。如果有shopt_option，-O 将设置那个选项的取值； +O 取消它。如果没有给出 shopt_option，shopt 将在标准输出上打印设为允许的选项的名称和值。如果启动选项是 +O，输出将以一种可以重用为输入的格式显示。
- --: --标志选项的结束，禁止其余的选项处理。任何 -- 之后的参数将作为文件名和参数对待。参数 - 与此等价。

Bash 也解释一些多字节的选项。在命令行中，这些选项必须置于需要被识别的单字符参数之前。

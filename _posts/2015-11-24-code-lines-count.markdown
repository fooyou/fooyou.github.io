---
layout: post
title: linux下代码行统计
category: Document
tags: linux
date: 2015-11-24 11:05:10
published: true
summary: Windows下有各种代码行统计工具，你的我的它的，linux下更是有各种方法工具。
image: pirates.svg
comment: true
latex: false
---

## 一、简单统计命令 find + wc

**用法一： 查找当前目录下的python文件总行数**

```
$ find . -name '*.c' | xargs wc -l
```

输出如下：

```
    ...
   592 ./lzma920/C/Util/SfxSetup/SfxSetup.c
    88 ./lzma920/C/Xz.c
    33 ./lzma920/C/XzCrc64.c
   875 ./lzma920/C/XzDec.c
   497 ./lzma920/C/XzEnc.c
   306 ./lzma920/C/XzIn.c
 30884 总用量
```

改进一下，只显示代码行：

```
$ find . -name '*.c' -print0 | xargs -0 cat | wc -l
```

输出如下：

```
30884
```

还有'*.h'文件呢，难道在来一遍？多个类型文件呢？再改进如下：

```
$ find . \( -name '*.h' -o -name '*.c' \) -print0 | xargs -0 cat | wc -l
```

或者，更直观的多类型文件统计：

```
$ find . '*.[h|c|cpp|cc|hpp]' -print0 | xargs -0 cat | wc -l
```

输出：

```
74575
```

我去，原来头文件还有这么多工作量啊？！

*空行不能算代码量*

*注释也不能算代码量（腹诽：为毛不算，程序员的注释是特么写小说的吗？那可是维护代码的财富，重要性不亚于代码本身）*

还是不够专业，于是下面的工具出场：

## 二、cloc、SLOCCount

专业的工具来了。

[cloc](http://cloc.sourceforge.net/), [SLOCCount](http://www.dwheeler.com/sloccount/)，介绍什么的略过，自己看文档。下面拿cloc为例看一下工具的效果。

1. 安装

```
$ sudo apt-get install cloc
```

2. 使用

统计7zip源码根目录下的代码量：

```
$ cloc .
```

结果：

```
    1512 text files.
    1179 unique files.
     223 files ignored.

http://cloc.sourceforge.net v 1.60  T=6.76 s (156.1 files/s, 30121.9 lines/s)
-------------------------------------------------------------------------------
Language                     files          blank        comment           code
-------------------------------------------------------------------------------
C++                            447          15589           6199         111225
C/C++ Header                   475           6759           2259          26356
C                               56           2427            722          19067
C#                              20            470            229           3846
make                            32            470              0           3715
Java                            14            428             22           3077
Assembly                         4            110             10            457
MSBuild scripts                  1              0              0             90
Teamcenter def                   6              6              0             60
-------------------------------------------------------------------------------
SUM:                          1055          26259           9441         167893
-------------------------------------------------------------------------------

```

很酷有没有，但sloccount有更酷的功能，它能统计代码量和开发效率，不知道它根据什么统计的，可能是git或者svn的提交记录，但若是没有版本库只有源码呢，时间有限没有尝试也没有看官方文档，但看结果：

```
Creating filelist for code
Creating filelist for lzma920
Have a non-directory at the top, so creating directory top_dir
Adding /media/joshua/My_Resource/Research/Open_Source/7-zip/./lzma920.tar.bz2 to top_dir
Categorizing files.
Finding a working MD5 command....
Found a working MD5 command.
Warning: in lzma920, number of duplicates=255
Computing results.


SLOC    Directory   SLOC-by-Language (Sorted)
131716  code            cpp=114807,ansic=16460,asm=449
32960   lzma920         cpp=20380,ansic=5657,cs=3846,java=3077
0       top_dir         (none)


Totals grouped by language (dominant language first):
cpp:         135187 (82.09%)
ansic:        22117 (13.43%)
cs:            3846 (2.34%)
java:          3077 (1.87%)
asm:            449 (0.27%)




Total Physical Source Lines of Code (SLOC)                = 164,676
Development Effort Estimate, Person-Years (Person-Months) = 42.51 (510.12)
 (Basic COCOMO model, Person-Months = 2.4 * (KSLOC**1.05))
Schedule Estimate, Years (Months)                         = 2.23 (26.72)
 (Basic COCOMO model, Months = 2.5 * (person-months**0.38))
Estimated Average Number of Developers (Effort/Schedule)  = 19.09
Total Estimated Cost to Develop                           = $ 5,742,532
 (average salary = $56,286/year, overhead = 2.40).
SLOCCount, Copyright (C) 2001-2004 David A. Wheeler
SLOCCount is Open Source Software/Free Software, licensed under the GNU GPL.
SLOCCount comes with ABSOLUTELY NO WARRANTY, and you are welcome to
redistribute it under certain conditions as specified by the GNU GPL license;
see the documentation for details.
Please credit this data as "generated using David A. Wheeler's 'SLOCCount'."
```

连每个开发者的花费都算出来了，着实，呵呵！！

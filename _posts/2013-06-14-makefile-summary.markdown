---
layout: post
title: Makefile 文件概要
category: Document
tags: make
date: 2021-09-24 12:09:07
published: true
summary: 通读本文可以手写简单的 Makefile
image: pirates.svg
comment: true
---

## 什么是 make

make 工具是构建依赖项的管理器。也就是说，它负责了解以何种顺序执行哪些命令才能从源文件、目标文件、库、头文件等的集合中获取软件项目（其中一些可能已最近更改）并将它们转换为程序的正确版本。

除此之外，也可以将 make 用到其他的地方，但不是本文讨论的内容。

## 一个简单的例子

假如有如下的文件结构，tool 依赖于 support，任务是编译可执行文件 tool

```bash
.
├── Makefile
├── support.cpp
├── support.h
└── tool.cpp
```

如果仅是一次性编译执行，那么仅仅：

```bash
$ g++ -o tool tool.cpp support.cpp
```

即可。每次都在命令行这么写肯定麻烦，所以可以把它写入 Makefile，然后每次 make 就可以了：

```makefile
tool:
	g++ -o tool.bin tool.cpp support.cpp
```

*tool 只是一个链接符号，可以为其他名称*

但在开发中我们的项目往往比较复杂，从头编译需要很长时间，对于某些没有变化的文件不需要编译，所以才有了.o 文件。用 g++ 命令来这个构建过程如下：

```bash
$ g++ -c support.cpp                # 用来检查 support.cpp 是否比 support.o 更新，如果是则进行编译，否则不进行编译
$ g++ -c tool.cpp
$ g++ -o tool.bin tool.o support.o
```

写成 Makefile 如下：

```makefile
tool: tool.o support.o
	g++ -o tool.bin tool.o support.o

tool.o:
	g++ -c tool.cpp

support.o:
	g++ -c support.cpp

```

makefile 的基本格式

```makefile
target: dependencies
	command
```

此处未缩进的行具有“目标：依赖项”的形式，并告诉 Make 如果任何依赖项比目标更新，则应运行关联的命令（缩进行）。也就是说，依赖行描述了需要重新构建以适应各种文件中的更改的逻辑。如果 support.cpp 更改意味着 support.o 必须重建，但 tool.o 可以不理会。当 support.o 更改时必须重建 tool。

## 变量、内置规则和其他好东西

看上面的例子，你会发现很生硬，但 make 是一种强大的语言，具有变量、文本操作函数和大量内置规则，可以让我们跟容易的做到构建工作。

### 变量

访问 make 变量的语法是 `$(VAR)`

定义和赋值：

```makefile
VAR = text value
VAR := something else
```

接下俩用变量改进上面的 makefile:

```makefile
CPPFLAGS=
LDFLAGS=-g
LDLIBS=

tool: tool.o support.o
	g++ $(LDFLAGS) -o tool.bin tool.o support.o $(LDLIBS)

tool.o:
	g++ $(CPPFLAGS) -c tool.cpp

support.o:
	g++ $(CPPFLAGS) -c support.cpp
```

## make 函数

GNU make 支持从文件系统或系统上的其他命令访问信息的各种函数。在这种情况下，我们感兴趣的是$(shell ...)which 扩展到参数的输出，以及$(subst opat,npat,text)替换文本中所有opatwith 的实例npat。

```makefile
CPPFLAGS=-g $(shell root-config --cflags)
LDFLAGS=-g $(shell root-config --ldflags)
LDLIBS=$(shell root-config --libs)

tool: tool.o support.o
	g++ $(LDFLAGS) -o tool.bin tool.o support.o $(LDLIBS)

tool.o:
	g++ $(CPPFLAGS) -c tool.cpp

support.o:
	g++ $(CPPFLAGS) -c support.cpp
```

目前为止：

1. 我们仍然明确说明每个目标文件和最终可执行文件的依赖关系
2. 我们必须明确输入两个源文件的编译规则

## 隐式规则和模式规则

我们通常希望所有 C++ 源文件都应该以相同的方式处理，Make 提供了三种方式来说明这一点：

1. 后缀规则（在 GNU make 中被认为已过时，但为了向后兼容而保留）
2. 隐式规则
3. 模式规则

隐式规则是内置的，下面将讨论一些。模式规则以类似的形式指定：

```makefile
*.o: %.c
	$(CC) $(CFLAGS) $(CPPFLAGS) -c $<
```

这意味着目标文件是通过运行显示的命令从 C 源文件生成的，其中“自动”变量`$<`扩展为第一个依赖项的名称。

## 内置规则

Make 有很多内置规则，这意味着实际上，一个项目通常可以由一个非常简单的 makefile 编译。

C 源文件的 GNU make 内置规则是上面展示的规则。同样，我们从 C++ 源文件创建目标文件，规则类似于$(CXX) -c $(CPPFLAGS) $(CFLAGS).

使用 链接单个目标文件$(LD) $(LDFLAGS) n.o $(LOADLIBES) $(LDLIBS)，但这在我们的情况下不起作用，因为我们想要链接多个目标文件。

## 内置规则使用的变量

内置规则使用一组标准变量，允许您指定本地环境信息（例如在哪里可以找到 ROOT 包含文件），而无需重新编写所有规则。我们最有可能感兴趣的是：

- `CC`: 要使用的 C 编译器
- `CXX`: 要使用的 C++ 编译器
- `LD`: 要使用的连接器
- `CFLAGS`: C 源文件的编译标志
- `CXXFLAGS`: C++ 源文件的编译标志
- `CPPFLAGS`: c 预处理器的标志（通常包括在命令行上定义的文件路径和符号），由 C 和 C++ 使用
- `LDFLAGS`: 连接器标志
- `LDLIBS`: 要链接的库


## 一个基础的 Makefile

通过内置规则，我们将 makefile 简化为：

```makefile
CC=gcc
CXX=g++
CPPFLAGS=-g $(shell root-config --cflags)
LDFLAGS=-g $(shell root-config --ldflags)
LDLIBS=$(shell root-config --libs)

SRCS=tool.cpp support.cpp
OBJS=$(subst .cpp,.o,$(SRCS))

all: tool

tool: $(OBJS)
	$(CXX) $(LDFLAGS) -o tool.bin $(OBJS) $(LDLIBS)

tool.o: tool.cpp support.h

support.o: support.h support.cpp

clean:
	$(RM) $(OBJS)
```

请注意，当不带参数调用 make 时，它使用文件中找到的第一个目标（在本例中为all），但您也可以命名要获取的目标，make clean在这种情况下，它是删除目标文件的原因。

我们仍然对所有依赖项进行了硬编码。

## 一些神秘改进

```makefile
C=gcc
CXX=g++
RM=rm -f
CPPFLAGS=-g $(shell root-config --cflags)
LDFLAGS=-g $(shell root-config --ldflags)
LDLIBS=$(shell root-config --libs)

SRCS=tool.cpp support.cpp
OBJS=$(subst .cpp,.o,$(SRCS))

all: tool

tool: $(OBJS)
	$(CXX) $(LDFLAGS) -o tool $(OBJS) $(LDLIBS)

depend: .depend

.depend: $(SRCS)
	$(RM) ./.depend
	$(CXX) $(CPPFLAGS) -MM $^>>./.depend;

clean:
	$(RM) $(OBJS)

distclean: clean
	$(RM) *~ .depend

include .depend
```

注意：

1. 源文件不再有任何依赖行！？！
2. .depend 和depend 有一些奇怪的魔法
3. 如果你这样做make，然后ls -A你看到一个文件名为.depend它包含的东西，看起来像化妆依赖行


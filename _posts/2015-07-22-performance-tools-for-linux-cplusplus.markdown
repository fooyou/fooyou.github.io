---
layout: post
title: C/C++性能测试工具 GNU gprof
category: Document
tags: c++ performance
year: 2015
month: 07
day: 22
published: true
summary: linux下的c/c++性能分析工具，用到了graphicviz绘图。
image: pirates.svg
comment: true
---

## 代码剖析（Code profiling）

程序员在优化软件性能时要注意应尽量优化软件中被频繁调用的部分，这样才能对程序进行有效优化。使用真实的数据，精确的分析应用程序在时间上的花费的行为就成为_代码剖析_。现在几乎所有的开发平台都支持代码剖析，本文要介绍的是linux下针对c/c++的GNU的gprof代码剖析工具。

> _PS：gprof不只能对c/c++，还可对Pascal和Fortran 77进行代码剖析。_

## gprof

GNU gprof 是一款linux平台上的程序分析软件（unix也有prof)。借助gprof可以获得C/C++程序运行期间的统计数据，例如每个函数耗费的时间，函数被调用的次数以及各个函数相互之间的调用关系。gprof可以帮助我们找到程序运行的瓶颈，对占据大量CPU时间的函数进行调优。

> _PS：gprof统计的只是用户态CPU的占用时间，不包括内核态的CPU时间。gprof对I/O瓶颈无能为力，耗时甚久的I/O操作很可能只占据极少的CPU时间。_

### 如何使用gprof

gprof的使用很简单，遵循以下步骤即可：

1. 使用编译标志`-pg`编译代码。
2. 运行程序生成剖析数据。
3. 运行gprof分析剖析数据，得到可视结果。

下面，我们来演练一下：

_test.c:_

```c
#include <stdio.h>

void func();

void a() {
    printf("Inside a()\n");
    int i = 0;

    for (; i < 0xffffff; ++i);
    func();
    return;
}

static void b() {
    printf("Inside b()\n");
    int i = 0;

    for (; i < 0xffffff; ++i);
    return;
}

int main() {
    printf("Inside main()\n");
    int i = 0;
    for (; i < 0xfffff; ++i);
    a();
    b();
    return 0;
}
```

> PS: for循环被用来产生执行时间。

_func.c:_

```c
#include <stdio.h>

void func() {
    printf("Inside func\n");
    int i = 0;

    for (; i < 0xffffff; ++i);
    return;
}
```

#### Step 1: 使用`-pg`标识编译上述代码

gcc文档中对`-pg`的描述：

> -pg : Generate extra code to write profile information suitable for the analysis program gprof. You must use this option when compiling the source files you want data about, and you must also use it when linking.

也就是，在编译和链接的时候都要使用`-pg`标识，所以，一起用吧：

```
$ gcc -Wall -pg test.c func.c -o test
```

#### Step 2: 运行程序

```
$ ls
func.c  makefile  test  test.c

$ ./test
Inside main()
Inside a()
Inside func
Inside b()

$ ls
func.c  gmon.out  makefile  test  test.c
```

这时，会发现目录下多了一个文件`gmon.out`，可以用gprof来分析它了。

#### Step 3: 使用gprof分析工具

gprof可以把gmon.out以人可读的方式解析出来，解析出的内容包括两个表（flat profile和call graph），一个包含函数执行时间，一个包含函数调用过程。

把这两个表重定向到analysis.txt：

```
$ gprof test gmon.out > analysis.txt
```

得到analysis.txt：

```
Flat profile:

Each sample counts as 0.01 seconds.
  %   cumulative   self              self     total           
 time   seconds   seconds    calls   s/call   s/call  name    
 33.69     10.42    10.42        1    10.42    10.42  b
 33.65     20.82    10.41        1    10.41    20.81  a
 33.65     31.23    10.41        1    10.41    10.41  func
  0.13     31.27     0.04                             main

 %         the percentage of the total running time of the
time       program used by this function.

cumulative a running sum of the number of seconds accounted
 seconds   for by this function and those listed above it.

 self      the number of seconds accounted for by this
seconds    function alone.  This is the major sort for this
           listing.

calls      the number of times this function was invoked, if
           this function is profiled, else blank.
 
 self      the average number of milliseconds spent in this
ms/call    function per call, if this function is profiled,
	   else blank.

 total     the average number of milliseconds spent in this
ms/call    function and its descendents per call, if this 
	   function is profiled, else blank.

name       the name of the function.  This is the minor sort
           for this listing. The index shows the location of
	   the function in the gprof listing. If the index is
	   in parenthesis it shows where it would appear in
	   the gprof listing if it were to be printed.

Copyright (C) 2012-2014 Free Software Foundation, Inc.

Copying and distribution of this file, with or without modification,
are permitted in any medium without royalty provided the copyright
notice and this notice are preserved.

		     Call graph (explanation follows)


granularity: each sample hit covers 2 byte(s) for 0.03% of 31.27 seconds

index % time    self  children    called     name
                                                 <spontaneous>
[1]    100.0    0.04   31.23                 main [1]
               10.41   10.41       1/1           a [2]
               10.42    0.00       1/1           b [3]
-----------------------------------------------
               10.41   10.41       1/1           main [1]
[2]     66.6   10.41   10.41       1         a [2]
               10.41    0.00       1/1           func [4]
-----------------------------------------------
               10.42    0.00       1/1           main [1]
[3]     33.3   10.42    0.00       1         b [3]
-----------------------------------------------
               10.41    0.00       1/1           a [2]
[4]     33.3   10.41    0.00       1         func [4]
-----------------------------------------------

 This table describes the call tree of the program, and was sorted by
 the total amount of time spent in each function and its children.

 Each entry in this table consists of several lines.  The line with the
 index number at the left hand margin lists the current function.
 The lines above it list the functions that called this function,
 and the lines below it list the functions this one called.
 This line lists:
     index	A unique number given to each element of the table.
		Index numbers are sorted numerically.
		The index number is printed next to every function name so
		it is easier to look up where the function is in the table.

     % time	This is the percentage of the `total' time that was spent
		in this function and its children.  Note that due to
		different viewpoints, functions excluded by options, etc,
		these numbers will NOT add up to 100%.

     self	This is the total amount of time spent in this function.

     children	This is the total amount of time propagated into this
		function by its children.

     called	This is the number of times the function was called.
		If the function called itself recursively, the number
		only includes non-recursive calls, and is followed by
		a `+' and the number of recursive calls.

     name	The name of the current function.  The index number is
		printed after it.  If the function is a member of a
		cycle, the cycle number is printed between the
		function's name and the index number.


 For the function's parents, the fields have the following meanings:

     self	This is the amount of time that was propagated directly
		from the function into this parent.

     children	This is the amount of time that was propagated from
		the function's children into this parent.

     called	This is the number of times this parent called the
		function `/' the total number of times the function
		was called.  Recursive calls to the function are not
		included in the number after the `/'.

     name	This is the name of the parent.  The parent's index
		number is printed after it.  If the parent is a
		member of a cycle, the cycle number is printed between
		the name and the index number.

 If the parents of the function cannot be determined, the word
 `<spontaneous>' is printed in the `name' field, and all the other
 fields are blank.

 For the function's children, the fields have the following meanings:

     self	This is the amount of time that was propagated directly
		from the child into the function.

     children	This is the amount of time that was propagated from the
		child's children to the function.

     called	This is the number of times the function called
		this child `/' the total number of times the child
		was called.  Recursive calls by the child are not
		listed in the number after the `/'.

     name	This is the name of the child.  The child's index
		number is printed after it.  If the child is a
		member of a cycle, the cycle number is printed
		between the name and the index number.

 If there are any cycles (circles) in the call graph, there is an
 entry for the cycle-as-a-whole.  This entry shows who called the
 cycle (as parents) and the members of the cycle (as children.)
 The `+' recursive calls entry shows the number of function calls that
 were internal to the cycle, and the calls entry for each member shows,
 for that member, how many times it was called from other members of
 the cycle.

Copyright (C) 2012-2014 Free Software Foundation, Inc.

Copying and distribution of this file, with or without modification,
are permitted in any medium without royalty provided the copyright
notice and this notice are preserved.

Index by function name

   [2] a                       [4] func
   [3] b                       [1] main

```

1. 使用`-a`参数屏蔽静态（私有）函数信息：

```
$ gprof -a test gmon.out > analysis.txt
```

2. 使用`-b`参数屏蔽冗余信息：

```
$ gprof -b test gmon.out > analysis.txt
```

得到如下信息：

```
Flat profile:

Each sample counts as 0.01 seconds.
  %   cumulative   self              self     total           
 time   seconds   seconds    calls   s/call   s/call  name    
 33.69     10.42    10.42        1    10.42    10.42  b
 33.65     20.82    10.41        1    10.41    20.81  a
 33.65     31.23    10.41        1    10.41    10.41  func
  0.13     31.27     0.04                             main


			Call graph


granularity: each sample hit covers 2 byte(s) for 0.03% of 31.27 seconds

index % time    self  children    called     name
                                                 <spontaneous>
[1]    100.0    0.04   31.23                 main [1]
               10.41   10.41       1/1           a [2]
               10.42    0.00       1/1           b [3]
-----------------------------------------------
               10.41   10.41       1/1           main [1]
[2]     66.6   10.41   10.41       1         a [2]
               10.41    0.00       1/1           func [4]
-----------------------------------------------
               10.42    0.00       1/1           main [1]
[3]     33.3   10.42    0.00       1         b [3]
-----------------------------------------------
               10.41    0.00       1/1           a [2]
[4]     33.3   10.41    0.00       1         func [4]
-----------------------------------------------


Index by function name

   [2] a                       [4] func
   [3] b                       [1] main

```

3. 使用`-p`参数只打印flat profile信息：

```
$ gprof -p test gmon.out > analysis.txt
```

4. 使用`-p(function)`参数只打印`function`函数信息：

只打印函数`a()`的flat profile信息

```
$ gprof -pa test gmon.out > analysis.txt
```

5. 使用`-P`参数屏蔽flat profile信息：

```
$ gprof -P test gmon.out > analysis.txt
```

6. 使用`-q`参数只打印call graph信息：

```
$ gprof -q test gmon.out > analysis.txt
```

7. 使用`-q(function)`参数只打印`function`函数的call graph信息：

只打印函数`a()`的call graph信息

```
$ gprof -qa test gmon.out > analysis.txt
```

8. 使用`-Q`参数屏蔽call graph信息：

```
$ gprof -Q test gmon.out > analysis.txt
```

可以组合使用这个参数。

### 参考：

> http://www.cnblogs.com/rocketfan/archive/2009/11/15/1603465.html

> http://www.thegeekstuff.com/2012/08/gprof-tutorial/

> http://blog.csdn.net/leichelle/article/details/8208530

> http://www.ibm.com/developerworks/cn/linux/l-gnuprof.html

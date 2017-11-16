---
layout: post
title: gdb 调试 cpp 代码
category: Document
tags: c++ debug gdb
date: 2016-04-12 09:04:38
published: true
summary: gdb c++ 入门级简单调试
image: pirates.svg
comment: true
---

切换到 linux 以来，作为一个 c/c++ 程序员首次回归到 c++ 有点小兴奋，尽管现在多使用脚本或 java，但回归到 c++ 仍然是十分惬意的。

多年前就学过 gdb 但一次也没有用过，忘光了，还好经常用 Python 脚本的 pdb 调试器，和 gdb 很相似。

**GDB 简介**


为使用 gdb 调试先写段有问题的求阶乘的代码：


```c++
$ vim factorial.c
#include <stdio.h>

int main()
{
    int i, num, j;
    printf("输入一个整数： ");
    scanf("%d", &num);
    for (i = 1; i < num; ++i)
    {
        j = j * i;
    }
    printf("%d 的阶乘是 %d", num, j);
}
```

好了，现在开始使用 gdb 调试

## Step 1: 使用 -g 参数编译代码

-g 参数可以使编译器收集调试信息：

```
$ cc -g factorial.c
```

上述命令会生成 a.out 文件，这个文件用于调试

## Step 2：使用 gdb 加载 a.out

```
$ gdb a.out
Reading symbols from a.out...done.
(gdb)
```

## Step 3：设置断点

语法：

```
(b)reak line_number
(b)reak [file_name]:line_number
(b)reak [file_name]:func_name
```

设置断点后，就可以在执行时中断并调试了

```
(gdb) b 10
Breakpoint 1 at 0x4005d1: file factorial.c, line 10.
```

## Step 4：在 gdb 中执行程序

语法：

```
run [args]
args: 执行时参数
```

执行

```
(gdb) run
Starting program: /media/joshua/My_Resource/Exercise/C++/gdb/a.out

Breakpoint 1, main () at factorial.c:10
10          for (i = 1; i <= num; ++i) {
(gdb)
```

接下来，就可以使用各种 gdb 命令调试了

## Step 5： 打印变量的值

语法：

```
(p)rint {variable}
```

举例：

```
(gdb) p i
$1 = 0
(gdb) p j
$2 = 0
(gdb) p num
$3 = 3
(gdb)

```

可看到 j 的值为 0（我去，以前的编译器不会自动初始化变量，从什么时候开始 gcc 会自动初始化变量了呢？），这导致结果一直为 0，所以就找到原因了，给 j 初始化为 1，然后继续调试，发现循环里应该 <= num，这就 OK 了。

## Step 6：继续执行和 Step in 和 Step over

- c / continue：继续执行直到下一个断点中断
- n / next：单步执行 Step out。
- s / step：单步执行，但若遇到函数则会进入函数内部，Step into。

    
其他命令：

- l / list：查看源代码段， l num 查看第 num 行上下文； l function 查看函数上下文
- bt / backtrace：打印执行堆栈或者最近的 frame 数
- h / help：帮助
- q / quit：退出

## 小提示：


### a). 修改变量

有时候调试时找到问题点了，想修改某个变量值来验证是不是好用，这样会节省很多时间，在 gdb里可以这么做：

1. 使用 set variable ：
    ```
    (gdb) set variable j = 1
    (gdb) p j
    $1 = 1
    ```
2. 直接修改变量内存地址中的内容：
    ```
    (gdb) set {int}0x7fffffffe37c = 0
    (gdb) p j
    $10 = 0
    ```

### b). 查看断点

查看所有断点

```
(gdb) info b
```

查看某个断点

```
(gdb) info b [n]
```

### c). 删除某个断点

```
clear：删除在选中 stack frame 即将执行的任何断点，最适合应用在中断后删除当前断点；
clear function
clear linenum
clear filename:linenum
```

## Hacker Staff

[gdbinit](https://github.com/gdbinit/Gdbinit) GDB 的配置文件，Hacker 的必备[dotgdb](https://github.com/dholm/dotgdb) 更棒的基于 gdbinit 的 gdb 配置文件，添加了众多实用的命令。


参考：

> http://www.thegeekstuff.com/2010/03/debug-c-program-using-gdb/

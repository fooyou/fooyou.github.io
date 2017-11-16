---
layout: post
title: 使用 Generator 创建新的迭代式样
category: Document
tags: python
date: 2016-01-26 16:01:33
published: true
summary: 创建自己的迭代式样
image: pirates.svg
comment: true
---

## 问题

如何编写一个自定义的样式的迭代器，比如和内建的 range() 或者 reversed() 函数不同的函数。

## 方案

如果要编写新种类的迭代样式，使用 generator 函数定义它，如下的 generator 将生产一定范围内的浮点数

```python
def frange(start, stop, increment):
    x = start
    while x < stop:
        yield x
        x += increment
```

要使用这个函数，可以使用 for 循环，或者像使用可迭代的函数比如 `sum()`和 `list()`一样。比如这样使用：

```python
for n in frange(0, 4, 0.2):
    print(n)
```

结果如下：

```
0
0.2
0.4
0.6000000000000001
0.8
1.0
1.2
1.4
1.5999999999999999
1.7999999999999998
1.9999999999999998
2.1999999999999997
2.4
2.6
2.8000000000000003
3.0000000000000004
3.2000000000000006
3.400000000000001
3.600000000000001
3.800000000000001
```

list()函数：

```python
list(frange(0, 1, 0.125))
```

结果如下：

```
[0, 0.125, 0.25, 0.375, 0.5, 0.625, 0.75, 0.875]
```

## 讨论

`yield`关键字把函数变成了一个生成器（generator），不像一个正常的函数，一个生成器函数只对迭代操作有返回值，下面将做个实验来揭示生成器函数是如何工作的：

```python
>>> def countdown(n):
...    print('Starting to count from', n)
...    while n > 0:
...        yield n
...        n -= 1
...    print('Done')

>>> # 创建生成器，注意并无输出。
>>> c = countdown(3)
>>> c
<generator object countdown at 0x7f4901806f78>

>>> # 执行到第一次 yield 并抛出一个值
>>> next(c)
Starting to count from 3
3

>>> # 执行到下一个 yield
>>> next(c)
2

>>> # 执行到下一个 yield
>>> next(c)
1

>>> # 执行到下一个 yield（迭代结束）
>>> next(c)
Done!
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```

生成器函数一个特性就是它只在迭代时的`next`操作有回应。一旦生成器函数返回，迭代就结束。然而，常用于迭代的`for`循环处理了这些细节，所以通常不用考虑。

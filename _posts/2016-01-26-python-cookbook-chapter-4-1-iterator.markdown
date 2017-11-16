---
layout: post
title: 使用迭代器
category: Document
tags: python
date: 2016-01-26 15:01:19
published: true
summary: 迭代器是 python 最强大的特性。在高级层你可以简单的使用迭代器遍历序列中的元素，但是迭代器的强大远不止这些，你可以创建自己的迭代器对象，应用迭代器式样，制作产生器函数，等等。
image: pirates.svg
comment: true
---

## 问题 

如何不使用 for 关键词，遍历一个序列

## 方案

手动遍历，可以使用`next()`函数，然后使用`StopIteration`来捕获异常，看看这个读取文件行的代码：

```python
with open('/etc/passwd') as f:
    try:
        line = next(f)
        print(line, end='')
    except StopIteration:
        pass
```

通常，`StopIteration`用来通知迭代结束。然而，如果你像上述代码手动调用的话，还可以得到一个迭代结束的值，如下：

```python
with open('/etc/passwd') as f:
    while True:
        line = next(f, None)
        if line is None:
            break
        print(line, end='')
```

## 讨论

通常码农都使用`for`来遍历各种存储序列，然而总有些时候，你会想精确的控制迭代过程，因此知道迭代器是如何工作的很有用。

如下代码阐释了迭代时发生了什么：

```python
tems = [1, 2, 3]
>>> it = iter(items)
>>> next(it)
1
>>> next(it)
2
>>> next(it)
3
>>> next(it)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
>>> 
```



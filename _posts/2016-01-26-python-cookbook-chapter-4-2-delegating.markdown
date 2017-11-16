---
layout: post
title: 代理迭代
category: Document
tags: python
date: 2016-01-26 16:01:22
published: true
summary: 
image: pirates.svg
comment: true
---

## 问题

加入你建立了一个自定义的容器对象，这个对象内部有 list 或者 tuple 或者其他可迭代的数据类型。你如何能让这个容器对象也能迭代。


## 方案

通常，你只需要定义`__iter__()`方法，并在里面实现对内部数据的迭代就可以了，比如：

```python
class Node:
    def __init__(self, value):
        self._value = value
        self._children = []

    def __repr__(self):
        return 'Node({!r})'.format(self._value)

    def add_child(self, node):
        self._children.append(node)

    def __iter__(self):
        return iter(self._children)

# Example

if __name__ == '__main__':
    root = Node(0)
    child1 = Node(1)
    child2 = Node(2)
    root.add_child(child1)
    root.add_child(child2)

    for ch in root:
        print(ch)

```

输出为：

```
Node(1)
Node(2)
```

这段代码中，`__iter__()`方法简单的把迭代请求转到了`_children`属性上

## 讨论

python 的迭代器协议需要`__iter__()`来返回一个特殊的迭代器对象，这个对象实现了一个`__next__()`方法来施行实际的迭代。这里使用的`iter()`函数简化了自定义迭代的操作，iter(s) 通过 `s.__iter__()` 方法返回当前的迭代器，就像同理的 `len(s)`调用了 `s.__len__()`

---
layout: post
title: 从 Python 到 NumPy
category: Document
tags: numpy ml
date: 2020-11-06 11:11:12
published: true
summary: 一本 NumPy 使用经验分享的书，如果你不是大拿级别的看了会受益的。
image: pirates.svg
comment: true
---

## 2 简介

numpy 的一切都是围绕着向量化的，所以你要熟悉这些概念，「向量」（vectors）、「数组」（arrays）、「视图」（views）、「」（ufuncs）。

 属性     | 描述
----------|-------------------------------
 ndim     | 数组的维数（尺度）
 shape    | 数组的大小。这是一个整数元组，表示每个维度中数组的大小。对于具有 n 行和 m 列的矩阵，shape 将是(n, m)
 size     | 数组的元素总数。等于 shape 元素的乘积
 dtype    | 数组中元素类型
 itemsize | 数组中每个元素的大小（以字节为单位）
 data     | 数组实际元素的缓冲区
 strides  | 遍历数组时每个维度中需要步进的字节数，可提升效率

```python
>>> import numpy as np
>>> Z = np.arange(9).reshape(3, 3).astype(np.float16)
>>> Z
array([[0., 1., 2.],
       [3., 4., 5.],
       [6., 7., 8.]], dtype=float16)
>>> Z.ndim
2
>>> Z.shape
(3, 3)
>>> Z.size
9
>>> Z.dtype
dtype('float16')
>>> Z.itemsize
2
>>> Z.data
<memory at 0x115385bb0>
>>> Z.strides
(6, 2)
```

## 3 array 的剖析

 **内容**

- [简介](###简介)
- [内存布局](###内存布局)
- [视图和拷贝](###视图和拷贝)
    - [直接和间接访问](####直接和间接访问)
    - [临时拷贝](####临时拷贝)
- [结论](###结论)


### 简介

正如[序言](#序言)中所解释的，你应该对 numpy 有一个基本的经验来阅读这本书。如果不是这样的话，你最好在回来之前先从初学者教程开始。因此，我将在这里简要介绍 numpy 数组的基本结构，特别是关于内存布局、视图、拷贝和数据类型。如果你想让你的计算受益于 numpy 哲学，它们是需要理解的关键概念。

让我们考虑一个简单的例子，在这个例子中，我们希望从具有 dtype 为 np.float32 的数组中清除所有值。如何编写最快的操作？下面的语法非常简明（至少对于那些熟悉 numpy 的人来说是这样），但是上面的问题要求找到最快的操作。

```python
>>> Z = np.ones(4*1000000, np.float32)
>>> Z[...] = 0
```

如果您更仔细地观察 dtype 和数组的大小，可以发现这个数组可以被转换（即查看）成许多其他“兼容”的数据类型。所谓兼容，我的意思是 Z.size\*Z.itemsize 可以除以新的 dtype itemsize。

```python
>>> timeit("Z.view(np.float16)[...] = 0", globals())
100 loops, best of 3: 2.72 msec per loop
>>> timeit("Z.view(np.int16)[...] = 0", globals())
100 loops, best of 3: 2.77 msec per loop
>>> timeit("Z.view(np.int32)[...] = 0", globals())
100 loops, best of 3: 1.29 msec per loop
>>> timeit("Z.view(np.float32)[...] = 0", globals())
100 loops, best of 3: 1.33 msec per loop
>>> timeit("Z.view(np.int64)[...] = 0", globals())
100 loops, best of 3: 874 usec per loop
>>> timeit("Z.view(np.float64)[...] = 0", globals())
100 loops, best of 3: 865 usec per loop
>>> timeit("Z.view(np.complex128)[...] = 0", globals())
100 loops, best of 3: 841 usec per loop
>>> timeit("Z.view(np.int8)[...] = 0", globals())
100 loops, best of 3: 630 usec per loop
```

有趣的是，清除所有值的简明方法并不是最快的。通过将数组转换为更大的数据类型，如 np.float64，我们得到了 25% 的速度系数。但是，通过将数组视为字节数组(np.int8)，我们得到了 50% 的速度因子。这种加速的原因可以在 numpy 内部机制和编译器优化中找到。这个简单的例子说明了 numpy 的哲学，我们将在下面的小节中看到。

### 内存布局

[numpy 文档](https://numpy.org/doc/stable/reference/arrays.ndarray.html)对类 ndarray 的定义非常明确：

> 类 ndarray 的实例由一个连续的一维计算机内存段（由数组或其他对象拥有）与索引方案（将N个整数映射到块中某个项的位置）组成。

换言之，数组主要是一个连续的内存块，其可以使用索引方案(indexing schema)访问。这样的索引方案又由 shape 和数据类型定义，这正是定义新数组时所需要的：

```python
Z = np.arange(9).reshape(3, 3).astype(np.int16)
```

此外，由于 Z 不是视图，我们可以推导出数组的跨距（[strides](https://numpy.org/doc/stable/reference/generated/numpy.ndarray.strides.html)），它定义了遍历数组时每个维度的字节数。

```python
>>> strides = Z.shape[1] * Z.itemsize, Z.itemsize
>>> strides
(6, 2)
>>> Z.strides
(6, 2)
```

有了这些信息，我们就知道如何访问特定的项（由索引元祖设计）更加准确的是，如何计算开始和结束的偏移量

```python
offset_start = 0
for i in range(ndim):
    offset_start += strides[i] * index[i]
offset_end = offset_start + Z.itemsize
```

可以用 [tobytes](https://docs.scipy.org/doc/numpy/reference/generated/numpy.ndarray.tobytes.html) 来验证一下上述偏移是否正确：

```python
>>> Z = np.arange(9).reshape(3, 3).astype(np.int16)
>>> index = 1, 1
>>> Z[index].tobytes()
b'\x04\x00'
>>> offset = 0
>>> for i in range(Z.ndim):
...     offset + = Z.strides[i]*index[i]
>>> print(Z.tobytes()[offset_start:offset_end]
b'\x04\x00'
```

这个数组实际上可以从不同的角度考虑（比如：布局）：

**元素布局(Item layout)**

```
               shape[1]
                 (=3)
            ┌───────────┐

         ┌  ┌───┬───┬───┐  ┐
         │  │ 0 │ 1 │ 2 │  │
         │  ├───┼───┼───┤  │
shape[0] │  │ 3 │ 4 │ 5 │  │ len(Z)
 (=3)    │  ├───┼───┼───┤  │  (=3)
         │  │ 6 │ 7 │ 8 │  │
         └  └───┴───┴───┘  ┘
```

**展平元素布局(Flattened item layout)**

```
┌───┬───┬───┬───┬───┬───┬───┬───┬───┐
│ 0 │ 1 │ 2 │ 3 │ 4 │ 5 │ 6 │ 7 │ 8 │
└───┴───┴───┴───┴───┴───┴───┴───┴───┘

└───────────────────────────────────┘
               Z.size
                (=9)
```

**内存布局(C order, big endian)**

```graph
                         strides[1]
                           (=2)
                  ┌─────────────────────┐

          ┌       ┌──────────┬──────────┐ ┐
          │ p+00: │ 00000000 │ 00000000 │ │
          │       ├──────────┼──────────┤ │
          │ p+02: │ 00000000 │ 00000001 │ │ strides[0]
          │       ├──────────┼──────────┤ │   (=2x3)
          │ p+04  │ 00000000 │ 00000010 │ │
          │       ├──────────┼──────────┤ ┘
          │ p+06  │ 00000000 │ 00000011 │
          │       ├──────────┼──────────┤
Z.nbytes  │ p+08: │ 00000000 │ 00000100 │
(=3x3x2)  │       ├──────────┼──────────┤
          │ p+10: │ 00000000 │ 00000101 │
          │       ├──────────┼──────────┤
          │ p+12: │ 00000000 │ 00000110 │
          │       ├──────────┼──────────┤
          │ p+14: │ 00000000 │ 00000111 │
          │       ├──────────┼──────────┤
          │ p+16: │ 00000000 │ 00001000 │
          └       └──────────┴──────────┘

                  └─────────────────────┘
                        Z.itemsize
                     Z.dtype.itemsize
                           (=2)
```

如果现在对 Z 取一个切片，那将得到一个基于数组 Z 的视图（view）：

```python
V = Z[::2, ::2]
```

这样的视图是用 shape、dtype 和 strides 指定的，因为 strides （的存在）不能再仅从 dtype 和 shape 推断（）。

**视图的元素布局**

```graph
               shape[1]
                 (=2)
            ┌───────────┐

         ┌  ┌───┬╌╌╌┬───┐  ┐
         │  │ 0 │   │ 2 │  │            ┌───┬───┐
         │  ├───┼╌╌╌┼───┤  │            │ 0 │ 2 │
shape[0] │  ╎   ╎   ╎   ╎  │ len(Z)  →  ├───┼───┤
 (=2)    │  ├───┼╌╌╌┼───┤  │  (=2)      │ 6 │ 8 │
         │  │ 6 │   │ 8 │  │            └───┴───┘
         └  └───┴╌╌╌┴───┘  ┘

```

**展平的元素布局**

```graph
┌───┬╌╌╌┬───┬╌╌╌┬╌╌╌┬╌╌╌┬───┬╌╌╌┬───┐       ┌───┬───┬───┬───┐
│ 0 │   │ 2 │   ╎   ╎   │ 6 │   │ 8 │   →   │ 0 │ 2 │ 6 │ 8 │
└───┴╌╌╌┴───┴╌╌╌┴╌╌╌┴╌╌╌┴───┴╌╌╌┴───┘       └───┴───┴───┴───┘
└─┬─┘   └─┬─┘           └─┬─┘   └─┬─┘
  └───┬───┘               └───┬───┘
      └───────────┬───────────┘
               Z.size
                (=4)

```

**内存布局（C order, big endian）**

```graph
              ┌        ┌──────────┬──────────┐ ┐             ┐
            ┌─┤  p+00: │ 00000000 │ 00000000 │ │             │
            │ └        ├──────────┼──────────┤ │ strides[1]  │
          ┌─┤    p+02: │          │          │ │   (=4)      │
          │ │ ┌        ├──────────┼──────────┤ ┘             │
          │ └─┤  p+04  │ 00000000 │ 00000010 │               │
          │   └        ├──────────┼──────────┤               │ strides[0]
          │      p+06: │          │          │               │   (=12)
          │            ├──────────┼──────────┤               │
Z.nbytes ─┤      p+08: │          │          │               │
  (=8)    │            ├──────────┼──────────┤               │
          │      p+10: │          │          │               │
          │   ┌        ├──────────┼──────────┤               ┘
          │ ┌─┤  p+12: │ 00000000 │ 00000110 │
          │ │ └        ├──────────┼──────────┤
          └─┤    p+14: │          │          │
            │ ┌        ├──────────┼──────────┤
            └─┤  p+16: │ 00000000 │ 00001000 │
              └        └──────────┴──────────┘

                       └─────────────────────┘
                             Z.itemsize
                          Z.dtype.itemsize
                                (=2)

```

### 视图和拷贝

视图和拷贝（Views and copies）对于优化数值计算是重要的概念。上一节中虽然进行了操作，但整个故事还是有点复杂。

#### 直接和间接访问

首先，需要区分索引 [indexing](https://docs.scipy.org/doc/numpy/user/basics.indexing.html#) 和花式索引 [fancy indexing](https://docs.scipy.org/doc/numpy/reference/arrays.indexing.html#advanced-indexing)。indexing 总是返回视图而 fancy indexing 返回拷贝。这种差异很重要，因为在第一种情况下，修改视图会修改源数组，而在第二种情况下则不会。

```python
>>> Z = np.zeros(9)
>>> Z_view = Z[:3]
>>> Z_view[...] = 1
>>> Z
[ 1.  1.  1.  0.  0.  0.  0.  0.  0.]
>>> Z = np.zeros(9)
>>> Z_copy = Z[[0,1,2]]
>>> Z_copy[...] = 1
>>> Z
[ 0.  0.  0.  0.  0.  0.  0.  0.  0.]
```

因此，如果您需要花式索引，最好保留一份花式索引的副本（尤其是计算复杂的索引）并使用它：

```python
>>> Z = np.zeros(9)
>>> index = [0,1,2]
>>> Z[index] = 1
>>> print(Z)
[ 1.  1.  1.  0.  0.  0.  0.  0.  0.]
```

如果你不确定索引的结果是视图还是拷贝，可以检查结果的 base 是什么，如果是 None 那么结果是拷贝：

```python
>>> Z = np.random.uniform(0,1,(5,5))
>>> Z1 = Z[:3,:]
>>> Z2 = Z[[0,1,2], :]
>>> print(np.allclose(Z1,Z2))
True
>>> print(Z1.base is Z)
True
>>> print(Z2.base is Z)
False
>>> print(Z2.base is None)
True
```

请注意，有些 numpy 函数在可能的情况下会返回视图（例如：[ravel](https://docs.scipy.org/doc/numpy/reference/generated/numpy.ravel.html)），而有些总是返回拷贝（例如：[flatten](https://docs.scipy.org/doc/numpy/reference/generated/numpy.ndarray.flatten.html#numpy.ndarray.flatten)）

```python
>>> Z = np.zeros((5, 5))
>>> Z.ravel().base is Z
True
>>> Z[::2. ::2].ravel().base is Z
False
>>> Z.flatten().base is Z
False
```

#### 临时拷贝

可以像上一节那样显式地创建拷贝，但最常见的情况是隐式创建中间拷贝。在对数组执行某些运算时，会出现这种情况：

```python
>>> X = np.ones(10, dtype=np.int)
>>> Y = np.ones(10, dtypr=np.int)
>>> A = 2*X + 2*Y
```

在上面的示例中，创建了三个中间数组。一个用于保存 2\*X 的结果，一个用于保存 2\*Y 的结果，最后一个用于保存 2\*X + 2\*Y 的结果。在这种特定情况下，数组足够小，这并没有真正的区别。但是，如果数组很大，那么您必须小心处理这些表达式，并想知道是否可以用不同的方法来处理。例如，如果只有最终结果很重要，而且之后不需要 X 或 Y，则另一种解决方案是：

```python
>>> X = np.ones(10, dtype=np.int)
>>> Y = np.ones(10, dtype=np.int)
>>> np.multiply(X, 2, out=X)
>>> np.multiply(Y, 2, out=Y)
>>> np.add(X, Y, out=X)
```

使用此替代解决方案，并未创建临时数组。问题是，还有许多其他情况需要创建此类拷贝，这会影响下面示例中所示的性能：

```python
>>> X = np.ones(1000000000, dtype=np.int)
>>> Y = np.ones(1000000000, dtype=np.int)
>>> timeit("X = X + 2.0*Y", globals())
100 loops, best of 3: 3.61 ms per loop
>>> timeit("X = X + 2*Y", globals())
100 loops, best of 3: 3.47 ms per loop
>>> timeit("X += 2*Y", globals())
100 loops, best of 3: 2.79 ms per loop
>>> timeit("np.add(X, Y, out=X); np.add(X, Y, out=X)", globals())
1000 loops, best of 3: 1.57 ms per loop
```

### 结论

最后，我们做一个练习。给定两个向量Z1和Z2。我们想知道Z2是否是Z1的视图，如果是，这个视图是什么？

```python
>>> Z1 = np.arange(10)
>>> Z2 = Z1[1:-1:2]
```

```graph
   ╌╌╌┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬╌╌
Z1    │ 0 │ 1 │ 2 │ 3 │ 4 │ 5 │ 6 │ 7 │ 8 │ 9 │
   ╌╌╌┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴╌╌
   ╌╌╌╌╌╌╌┬───┬╌╌╌┬───┬╌╌╌┬───┬╌╌╌┬───┬╌╌╌╌╌╌╌╌╌╌
Z2        │ 1 │   │ 3 │   │ 5 │   │ 7 │
   ╌╌╌╌╌╌╌┴───┴╌╌╌┴───┴╌╌╌┴───┴╌╌╌┴───┴╌╌╌╌╌╌╌╌╌╌
```

首先，需要检查 Z2 的 base 是否是 Z1

```python
>>> Z2.base is Z1
True
```

此时，我们知道 Z2 是 Z1 的视图，这意味着 Z2 可以表示为 Z1 [start:stop:step]。困难在于找到起点、终点和步长。对于步长，我们可以使用任何数组的 strides 属性，该属性给出每个维度中从一个元素到另一个元素所需的字节数。在我们的例子中，由于两个数组都是一维的，我们可以直接比较第一个 strides：

```python
>>> step = Z2.strides[0] // Z1.strides[0]
>>> step
2
```

下一个困难是找到开始和结束位置。为此，我们可以利用 byte\_bounds 方法返回指向数组端点的指针。

```graph
  byte_bounds(Z1)[0]                  byte_bounds(Z1)[-1]
          ↓                                   ↓
   ╌╌╌┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬╌╌
Z1    │ 0 │ 1 │ 2 │ 3 │ 4 │ 5 │ 6 │ 7 │ 8 │ 9 │
   ╌╌╌┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴╌╌

      byte_bounds(Z2)[0]      byte_bounds(Z2)[-1]
              ↓                       ↓
   ╌╌╌╌╌╌╌┬───┬╌╌╌┬───┬╌╌╌┬───┬╌╌╌┬───┬╌╌╌╌╌╌╌╌╌╌
Z2        │ 1 │   │ 3 │   │ 5 │   │ 7 │
   ╌╌╌╌╌╌╌┴───┴╌╌╌┴───┴╌╌╌┴───┴╌╌╌┴───┴╌╌╌╌╌╌╌╌╌╌

```

```python
>>> offset_start = np.byte_bounds(Z2)[0] - np.byte_bounds(Z1)[0]
>>> offset_start
8
>>> offset_stop = np.byte_bounds(Z2)[-1] - np.byte_bounds(Z1)[-1]
offset_stop
-16
```

使用 itemsize 并考虑到 offset\_stop 为负（Z2 的结束边界在逻辑上小于 Z1 数组的结束边界），将这些偏移转换为索引非常简单。因此，我们需要加上 Z1 的元素大小来获得右端索引。

```python
>>> start = offset_start // Z1.itemsize
>>> stop = Z1.size + offset_stop // Z1.itemsize
>>> start, stop, step
(1, 8, 2)
```

最后我们测试一下结果：

```python
>>> np.allclose(Z1[start:stop:step], Z2)
True
```

作为一个练习，您可以通过考虑以下因素来改进第一个简明的实现：

- 负步长
- 多维数组

这个练习的[解决方案](https://www.labri.fr/perso/nrougier/from-python-to-numpy/code/find_index.py)


> http://www.labri.fr/perso/nrougier/from-python-to-numpy

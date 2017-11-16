---
layout: post
title: 1.2 关键词的差异
category: Document
tags: c++
date: 2016-02-19 10:02:34
published: true
summary: struct 和 class 关键词的差异
image: pirates.svg
comment: true
---

## 1.2 关键词的差异

C++ 面对对象的封装设计在开始时对 C 的一些过于繁杂的地方是有些抵触的，比如八种整形数的支持，但是为了完全的兼容 C，它不得不对这些旧的关键字和规范进行兼容，关键字 struct 和 class 的纠结也因此而生，经历了纠结之后，目前的 C++ 标准，其没有任何区别，但各家的语义分析器是否把这两货看成是一个东西？应该是的。


不管怎么说由于 C++ 的对象模型的引入，类的成员的声明次序和类对象内存分配是有影响的，类的 public、protected 和 private 这些 access section 内的数据，必须保证以其声明次序出现在内存布局中，但是跨 section 的数据的内存次序就不一定怎么排列了，这使得 C 语言的一些内存分配的小技巧有可能在 C++ 中玩不转。

同样的，base class 和 derived class 的 data members 的布局也没有谁先谁后的强制规定。


## 1.3 对象的差异（A Object Distinction）

C++ 程序设计直接支持三种 programming paradigms：

- 程序模型（procedural model）
- 抽象数据类型模型（abstract data type model, ADT）
- 面向对象模型

面向对象的设计一个特点就是多态性，C++ 有几种支持多态的方法：

- 隐含转化

    ```cpp
    shape *ps = new circle();
    ```

- virtual function 机制：

    ```cpp
    ps->rotate();
    ```

- dynamic_cast 和 typeid 运算符：

    ```cpp
    if (circle *pc = dynamic_cast <circle*> (ps) ) {}
    ```

多态的一个好处是减少代码变动，在程序执行期之前，无法决定调用哪个类的方法。

### 一个类对象占用多大空间？

- 其 nonstatic data members 的总和大小
- 加上由于 bus “运输量” 需要补齐的位数（比如 32bit 机器上需要 4 位置补齐）
- 加上为了支持 virtual 而内部产生的任何额外负担


### 指针类型

指向类对象的指针和指向整形数的指针有什么不同呢？

本质上没有不同，其差异不在其指针表示法不同，也不在其寻址内容（代表一个地址）不同，而是在其所寻址出来的对象类型不同。“指针类型”会教导编译器如何解释某特定地址中的内存内容及其大小。这一点在 void* 指针和其他指针转换时有明确的体现。

指针和多态，在每一个执行点（slot），决定多态函数调用的信息存储在 link 中，这个 link 存在 object 的 vptr 和 vptr 所指向的 virtual table 之间。


分析一段代码：

```cpp
Bear b;
ZooAnimal za = b; // 会引起切割（sliced）

// 调用 ZooAnimal::rotate()， base class 的 rotate 方法是 virtual 的
za.rotate();
```

可以看到 za.rotate 没有调用 Bear 的方法，原因如下：

- 在类对象初始化和赋值操作时，编译器必须保证如果某个  object 含有一个或一个以上的 vptrs，那些 vptrs 的内容不会被 base class object 初始化或改变。所以 za 赋值构造的时候不会拷贝 b 的 vptrs。
- 多态的潜在力量不能发挥在“直接存取 objects”上，OO 程序设计并不支持对 object 的直接处理。



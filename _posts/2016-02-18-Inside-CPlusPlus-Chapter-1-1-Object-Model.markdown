---
layout: post
title: Inside C++ Object 1.1 C++ 对象模式
category: Document
tags: c++
date: 2016-02-18 15:02:00
published: true
summary: C++对象模型的复习
image: pirates.svg
comment: true
---

好多年以前学习过候杰老师翻译的《深度探索C++对象模型》，现在抽空复习一边。

C语言中“数据”和“函数”之间无关联性，所以C是程序性的，其由一组“分布在各个以功能为导向的函数中”的算法驱动。

C++加上封装后相比C布局成本增加了多少？

- 成员函数虽然在类的声明内，但不会出现在对象中；
- 非内联成员函数只会诞生一个函数实体，内联函数会在每一个使用者（模块）身上产生一个函数实体；

所以比较结构体和C++对象，实现同样的功能，C++对象几乎没有增加额外的成本。

C++在布局和存取时间上主要的额外负担是由 virtual 引起的，包括：

- virtual function 虚拟函数机制：用来支持一个有效率的“执行期绑定”（runtime binding）；
- virtual base class：用来实现“多次出现在继承体系中的 base class，有一个单一而被共享的实体”；

还有一些多重继承下的额外负担，发生在 derived class 和 base class 的转换上。

## 1.1 C++ 对象模式

C++中有两种 class data members：

- staic
- nonstatic

三种 class member functions：

- staic
- nonstatic
- virtual

## C++ 对象模型

- Nonstatic data members 被配置于每个类对象之内；
- Static data members 被存放在所有类对象之外；
- Static 和 nonstatic function members 都被放在所有类对象之外；
- Virtual functions：
    - 每个类产生出一堆指向 virtual functions 的指针，放在 virtual table（vtbl）中
    - 每个类对象被添加一个指向 vtbl 的指针（vpbr），其设定和重置由每一个 class 的 constructor、destructor 和 copy assignment 运算符自动完成。
    - 每个类所关联的用来支持 runtime type identification 的 type_info object 也由 vtbl 被指出来，通常放在表格的第一个 slot 处。

**加上继承**

C++ 单一继承：

```C++
class ios {};
class istream : public ios {};
class iostream : public istream {};
```

C++ 多重继承：

```C++
class iostream : 
    public istream,
    public ostream {};
```

C++ 虚拟继承：

```C++
class istream : virtual public ios {};
class ostream : virtual public ios {};
```

虚拟继承下， base class 不管在继承串链中被派生多少次，永远只会存在一个实体。


---
layout: post
title: Effective C++ 复习
category: Document
tags: cpp
date: 2016-08-12 13:08:17
published: true
summary: 要跳槽了，复习一下 C++。
image: pirates.svg
comment: true
---

# C 到 C++

C 是简单的语言，真正提供的只有宏、指针、结构、数组和函数。C++ 中 C 的这些特性仍存在，但多了私有和保护成员、函数重载、缺省参数、构造和析构函数、自定义操作符、内联函数、引用、友元、模板、异常、命名空间等等。当然这只是能直观感受到的，然而在设计思想上 C 和 C++ 是不同的。

条款1：尽量用 const 和 inline 而不用 #define
---

> “尽量用编译器而不用预处理”

使用 const 和 inline，减少对预处理的需要，但抛弃 #include 的日子还很远，预处理还不能退休。

```cpp
#define ASPECT_RATIO 1.653
```

最好写成：

```cpp
const double ASPECT_RATIO = 1.653;
```

因为#define 的语句被预处理过，到编译器时看不到这条语句，如果编译报错不会指向 define 语句，会令人费解

```cpp
char greeting[] = "Hello";
char* p = greeting;         // 非 const 指针， 非 const 数据
const char* p = greeting;   // 非 const 指针， const 数据
char* const p = greeting;   // const 指针， 非 const 数据
const char* const p = greeting; // 指针数据都是 const 的

// 迭代器的 const 修饰
std::vector<int> vec;
const std::vector<int>::iterator iter = vec.begin();    // iter 的作用像个 T* const
*iter = 10;     // OK，const 修饰的是指针
++iter;         // Err

// std 中封装了 const_iterator
std::vector<int>::const_iterator cIter = vec.begin();
*cIter = 10;    // Err
++cIter;        // OK
```

const 的最具威力的用法是面对函数声明时的应用。可修饰函数返回值。

```cpp
class Rational {};
const Rational operator* (const Rational& lhs, const Rational& rhs);
const Rational operator* (const Rational& lhs, const Rational& rhs) const;  // 第一个 const 修饰返回值，后面的 const 修饰成员函数，意指：函数内不可修改非静态成员变量的值。
```

为什么要返回一个 const 对象，因为如果不这样客户可以实现这样的暴行：

```cpp
Rational a, b, c;
(a * b) = c;
```

一个重要的 C++ 特性：

**两个成员函数如果只是常量性(constness)不同，可以被重载**

mutable 关键字修饰的成员变量允许在 const 函数内修改。

const_cast 可以为指针去掉 const 修饰符；

static_cast 可以为指针加上 const 修饰符；

可以使用 const 成员函数来实现 non-const 同名成员函数，反之就违背了 const 函数的设计原则。

请记住：

- 将某些东西声明为 const 可帮助编译器侦测出错误用法。const 可被施加于任何作用域内的对象、函数参数、函数返回类型、成员函数本体。
- 编译器强制实施 bitwise constness，但你编写程序时应该使用“概念上的常量性”（conceptual constness）。
- 当 const 和 non-const 成员函数有着实质等价的实现时，令 non-const 版本调用 const 版本可避免代码重复。

### 条款2：尽量用 <iostream> 而不用 <stdio.h>

> "类型安全和扩展性是 C++ 的基石"

所以在写 C++ 代码的时候要时刻考虑到这一点，因为轻巧高效的 scanf 和 printf 是非类型安全的，又没有扩展性，还要把读写的变量和控制读写的格式信息分开来。这这些缺点正是操作符 << 和 >> 的强项。


### 条款3：尽量用 new 和 delete 而不用 malloc 和 free

malloc 和 free 太简单，不知道构造函数和析构函数


### 条款4：确定对象被使用前已先被初始化

Make sure that objects are initialized before they're used.

这里要注意初始化（initializations）和赋值（assignments）这两个概念：

```cpp
class ABEntry {
public:
    ABEntry(const std::string& name);
private:
    std::string theName;
};

ABEntry::ABEntry(const std::string& name)
{
    theName = name;     // 这是赋值非初始化。
}

ABEntry::ABEntry(const std::string& name)
: theName(name)         // 这是初始化
{
}
```

初始化的时序发生在 default 构造函数被自动调用时（比进入构造函数本体的时间要早），初始化比赋值的效率要高。

non-local static 对象的初始化次序：

static 对象：被构造出来直至程序结束，包括 global 对象、定义于 namespace 作用域内的对象、在 classes 内、在函数内、以及在 file 作用域被声明为 static 的对象。

local static 对象：函数内的 static 对象，其他的成为 non-local static 对象。



# 内存管理

### 条款5：对应的 new 和 delete要采用向同方的形式

尽量杜绝使用 typedef 定义数组类型，这样容易引起混乱，例如：

```cpp
typedef string AddressLine[4];
string *pal = new AddressLine;
delete [] pal;  // 正确
delete pal;     // 错误，会引起无法预料的错误。
```

### 条款6：析构函数里对指针成员调用 delete

delete 空指针是安全的

### 条款7：预先准备好内存不足的情况

operator new 在无法分配内存时会抛出异常，问题是你会处理这个异常吗？如果不处理，你心里一定会深深地隐藏着一种罪恶感。

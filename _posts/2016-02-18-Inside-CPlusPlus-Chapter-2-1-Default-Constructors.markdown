---
layout: post
title: 2.1 Default Constructor 的建构操作
category: Document
tags: c++
date: 2016-02-19 15:02:08
published: true
summary: 构造语意学（The Semantics of Constructors）
image: pirates.svg
comment: true
---

学习 C++ 一个听烂了的词叫 “C++ 编辑器会...” C++ 编辑器确实做了很多的事情，本章将展开在构造函数时它都干了些什么事。

## 2.1 Default Constructor 的建构操作

在默认构造上，程序员没干的活，编译器会干不干不行的活，该程序员干的活编译器不干（有好多事情比如初始化感觉就应该是编译器干的活`）。概念：

- implicit default constructor
- trivial default constructor
- nontrivial default constructor


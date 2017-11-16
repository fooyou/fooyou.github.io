---
layout: post
title: Aho-Corasick 算法
category: Document
tags: algorithm 
date: 2017-03-22 10:03:01
published: true
summary: 
image: pirates.svg
comment: true
---
## 简介

Aho-Corasick算法简称AC算法，通过将模式串预处理为确定有限状态自动机，扫描文本一遍就能结束。其复杂度为O(n)，即与模式串的数量和长度无关。

### 思想

自动机按照文本字符顺序，接受字符，并发生状态转移。这些状态缓存了“按照字符转移成功（但不是模式串的结尾）”、“按照字符转移成功（是模式串的结尾）”、“按照字符转移失败”三种情况下的跳转与输出情况，因而降低了复杂度。

### 基本构造

AC算法中有三个核心函数，分别是：

- success; 成功转移到另一个状态（也称goto表或success表）
- failure; 不可顺着字符串跳转的话，则跳转到一个特定的节点（也称failure表），从根节点到这个特定的节点的路径恰好是失败前的文本的一部分。
- emits; 命中一个模式串（也称output表）

## 举例

以经典的ushers为例，模式串是he/ she/ his /hers，文本为“ushers”。构建的自动机如图：

![自动机构建图](https://github.com/fooyou/fooyou.github.io/blob/master/img/dots/aho-corasick.dot.png?raw=true)

## 匹配过程

### 自动机从根节点0出发

1. 首先尝试按success表转移（图中实线）。按照文本的指示转移，也就是接收一个u。此时success表中并没有相应路线，转移失败。

2. 失败了则按照failure表回去（图中虚线）。按照文本指示，这次接收一个s，转移到状态3。

3. 成功了继续按success表转移，直到失败跳转步骤2，或者遇到output表中标明的“可输出状态”（图中红色状态）。此时输出匹配到的模式串，然后将此状态视作普通的状态继续转移。

算法高效之处在于，当自动机接受了“ushe”之后，再接受一个r会导致无法按照success表转移，此时自动机会聪明地按照failure表转移到2号状态，并经过几次转移后输出“hers”。来到2号状态的路不止一条，从根节点一路往下，“h→e”也可以到达。而这个“he”恰好是“ushe”的结尾，状态机就仿佛是压根就没失败过（没有接受r），也没有接受过中间的字符“us”，直接就从初始状态按照“he”的路径走过来一样（到达同一节点，状态完全相同）。

## 构造过程

success、failure 和 output 表是怎么被计算出来的呢？

### goto 表

很简单，了解一点trie树知识的话就能一眼看穿，goto表就是一棵trie树。把上图的虚线去掉，实线部分就是一棵trie树了。

![自动机构建图](https://github.com/fooyou/fooyou.github.io/blob/master/img/dots/aho-corasick-goto.dot.png?raw=true)

### output 表

output表也很简单，与trie树里面代表这个节点是否是单词结尾的结构很像。不过trie树只有叶节点才有“output”，并且一个叶节点只有一个output。下图却违背了这两点，这是为什么呢？其实下图的output会在建立failure表的时候进行一次拓充。

```
    i           output(i)
    2           {he}
    5           {she, he}
    7           {his}
    9           {hers}
```

以上两个表通过一个dfs就可以构造出来。关于trie树的更详细内容，请参考：

> [Ansj 分词双数组 Trie 树实现与 arrays.dic 词典格式](http://www.hankcs.com/nlp/ansj-word-pairs-array-tire-tree-achieved-with-arrays-dic-dictionary-format.html)
> [Trie 树分词](http://www.hankcs.com/program/java/tire-tree-participle.html)
> [双数组trie树doublearraytriejava实现](http://www.hankcs.com/program/java/双数组trie树doublearraytriejava实现.html)

### failure 表

这个表是trie树没有的，加了这个表，AC自动机就看起来不像一棵树，而像一个图了。failure表是状态与状态的一对一关系，别看图中虚线乱糟糟的，不过你仔细看看，就会发现节点只会发出一条虚线，它们严格一对一。

这个表的构造方法是：

1. 首先规定与状态0距离为1（即深度为1）的所有状态的fail值都为0。

2. 然后设当前状态是S1，求fail(S1)。我们知道，S1的前一状态必定是唯一的（刚才说的一对一），设S1的前一状态是S2，S2转换到S1的条件为接受字符C，测试S3 = goto(fail(S2), C)。

3. 如果成功，则fail(S1) = goto(fail(S2), C) = S3。

4. 如果不成功，继续测试S4 = goto(fail(S3), C)是否成功，如此重复，直到转换到某个有效的状态Sn，令fail(S1) = Sn。

```
    i       1 2 3 4 5 6 7 8 9
    f(i)    0 0 0 1 2 0 3 0 3
```

## 实现

Java 实现：

> https://github.com/hankcs/aho-corasick

调用方式：

```java
Trie trie = new Trie();
trie.addKeyword("hers");
trie.addKeyword("his");
trie.addKeyword("she");
trie.addKeyword("he");
Collection<Emit> emits = trie.parseText("ushers");
System.out.println(emits);
```

输出：

```
[2:3=he, 1:3=she, 2:5=hers]
```

本文参考：

> http://www.hankcs.com/program/algorithm/implementation-and-analysis-of-aho-corasick-algorithm-in-java.html


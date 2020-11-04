---
layout: post
title: 算法复杂度的符号（Θ、Ο、ο、Ω、ω）简介
category: Document
tags: algorithm
date: 2020-11-02 11:11:58
published: true
summary: 解析算法复杂度符号的含义，读论文或看文章时经常发现不同的复杂度符号，来下详细整理
image: pirates.svg
comment: true
---


 字母 | 注音   | 含义
------|--------|-------------------------------
 Θ    | theta  | 既是上届也是下届，等于的意思
 Ο    | oh     | 大写，表示上届，小于等于的意思
 ο    | oh     | 小写，表示上届，小于的意思
 Ω    | omega  | 大写，表示下届，大于等于的意思
 ω    | omega  | 小写，表示下届，大于的意思


 Letter     | Bound        | Grouth
------------|--------------|-------------------------------
 Θ(theta)   | upper and lower, tight | equal
 Ο(oh big)  | upper, tightness unknown | less than or equal
 ο(oh small)| upper not tight | less than
 Ω(omega big)| lower tightness unknown | greater than or equal
 ω(omega small) | lower not tight | greater than

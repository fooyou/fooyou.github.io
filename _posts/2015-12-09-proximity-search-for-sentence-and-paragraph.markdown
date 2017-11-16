---
layout: post
title: 使用lucene做句子和段落的同在查询
category: Document
tags: lucene
date: 2015-12-09 16:12:36
published: true
summary: lucene支持的Query里面有Proximity查询，却没有同句或同在的查询，要实现同在查询的话就得自己扩展QueryParser了。
image: pirates.svg
comment: true
---

调查整理了一番，主要有以下几种实现方案：

1. 使用Solr的正则

    参考：

    > http://stackoverflow.com/questions/364301/solr-using-regex-fragmenter-to-extract-paragraphs

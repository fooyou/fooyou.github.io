---
layout: post
title: 图数据库：存储半结构化大数据的解决方案
category: Document
tags: bigdata
date: 2016-12-09 10:12:49
published: true
summary: 几种数据库存储方案的对比
image: pirates.svg
comment: true 
---

数据变大，数据不够结构化，数据间有千丝万缕的连接，如何建立数据模型？

它们是怎么玩的：

- Facebook Graph
- LinkedIn Graph
- Linked Data
- Blogs/Tagging

## NOSQL 数据库

     类型       |     举例
----------------|--------------------
 Key-Value 存储 | redis、riak、Voldemort. Dynamo
 宽列存储       | HBase、Cassandra
 文件存储       | mongdb、CouchDB
 图存储         | Neo4j、titan、OrientDB


### Key-Value 存储

**数据模型**

- 全局 K-V 映射
- 大伸缩 HashMap
- 高容错 Highly fault tolerant

**优缺点**

- 简单的数据模型
- 可伸缩
- 但需要创建自己的“外键”
- 处理复杂数据乏力


### 列存储家族

基于 BigTable（Google 的对结构化数据分布式存储模型）的存储模型。

**数据模型**

- 大表,列族
- 用 Map Reduce 来检索和处理

**优缺点**

- 支持半结构化数据
- 天生索引能力（对列）
- 可伸缩
- 但对內连数据乏力（Interconnected data）

### 文档数据集

**数据模型**

- 文档集合
- 文档是 K-V 集合
- 使用 map-reduce 集中索引

**优缺点**

- 简单强大的数据模型
- 可伸缩
- 但对內连数据乏力
- 检索模型受限于 key 和 index
- 大数据量查询能力弱（Map reduce for larger queries）

### 图数据集

**数据模型**

节点和关系

**优缺点**

- 强大的数据模型，和 RDBMS 一样
- 关联数据本地索引
- 查询简单
- 但碎片化存储
- 需要不同的数据模型

## 什么是图

图就是一群连接在一起的对象集合的抽象表现

    对象 Object（Vertrx, Node）
    连接 Link(Edge, Arc, Relationship)

## 什么是图数据库

使用图结构做为数据模型的数据库，每个节点通过边可看到它的相邻节点，当节点增加，本地查询的复杂度不变。

## Apache Tinkerpop

处理如此复杂的数据里需要数据库提供方提供完备的 API 接口，但若是使用某一个数据库提供方提供的接口，当迁移至其他数据库时就麻烦的很，为解决这个问题，Tinkerpop 应运而生。

Tinkerpop 是图数据库的一般性API，它整合了 Titan DB, Neo4j, Orient DB 等等数据库

- 图处理系统
- 有 tinkerpop 3 结构化 API
    - Graph, Element, Property
- 有 tinkerpop 3 处理 API
    - TraversalSource, GraphComputer
- Gremlin 查询语言
- REST API

## Titan 数据库

使用 Apache Tinkerpop API 优化了分布式存储和检索包含上亿个顶点和边的可伸缩的图数据库，它支持 Apache Spark 和 Hadoop（默认）的 map-reduce 操作。

并集成了 ElasticSearch，Solr 和 Lucene，并使用如下后端存储：

- Apache Cassandra
- Apache HBase
- Oracle BerkeleyDB

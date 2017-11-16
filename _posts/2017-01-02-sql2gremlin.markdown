---
layout: post
title: gremlin 的查询语法
category: Document
tags: gremlin
date: 2017-01-02 17:01:50
published: true
summary: titan 使用 gremlin 查询体验
image: pirates.svg
comment: true 
---

使用 titan + hbase + elasticsearch

hbase: hadoop 集群

es: es 集群

配置 'conf/titan-hbase-elasticsearch.properties' 增加 hbase 和 es 的 table 和 index-name 名：

```
storage.hbase.table=hbase_es_test
index.search.index-name=hbase_es_test
```
打开 gremlin.sh :

```
$ ./bin/gremlin.sh
```

显示如下：

```
         \,,,/
         (o o)
-----oOOo-(3)-oOOo-----
plugin activated: aurelius.titan
plugin activated: tinkerpop.server
plugin activated: tinkerpop.utilities
15:51:46 INFO  org.apache.tinkerpop.gremlin.hadoop.structure.HadoopGraph  - HADOOP_GREMLIN_LIBS is set to: /Users/liuchaozhen/Downloads/titan-1.0.0-hadoop1/lib
plugin activated: tinkerpop.hadoop
plugin activated: tinkerpop.tinkergraph
gremlin>
```

加载配置项，创建 graph (不知道为啥出现了一个hdfs的异常，没有处理这个异常，但不影响后续操作)

```
gremlin> graph = TitanFactory.open('conf/titan-hbase-es.properties')
15:52:36 WARN  org.apache.hadoop.hbase.util.DynamicClassLoader  - Failed to identify the fs of dir hdfs://local
host:9000/hbase/lib, ignored
java.net.ConnectException: Call to localhost/127.0.0.1:9000 failed on connection exception: java.net.ConnectExc
eption: Connection refused
        at org.apache.hadoop.ipc.Client.wrapException(Client.java:1142)
        at org.apache.hadoop.ipc.Client.call(Client.java:1118)
		...
15:53:20 WARN  com.thinkaurelius.titan.graphdb.configuration.GraphDatabaseConfiguration  - Local setting cache.
db-cache-time=180000 (Type: GLOBAL_OFFLINE) is overridden by globally managed value (10000).  Use the Managemen
tSystem interface instead of the local configuration to control this setting.
15:53:20 WARN  com.thinkaurelius.titan.graphdb.configuration.GraphDatabaseConfiguration  - Local setting cache.
db-cache-clean-wait=20 (Type: GLOBAL_OFFLINE) is overridden by globally managed value (50).  Use the Management
System interface instead of the local configuration to control this setting.
==>standardtitangraph[hbase:[10.10.113.192, 10.10.113.193, 10.10.113.194]]
gremlin>
```

获取图的 traversal：

```
gremlin> g = graph.traversal()
==>graphtraversalsource[standardtitangraph[hbase:[10.10.113.192, 10.10.113.193, 10.10.113.194]], standard]
```

好，下面开始查询。

## Select

### Select all

```
gremlin> g.V().hasLabel('category').valueMap()
```

限制显示个数使用 limit(n):

```
gremlin> g.V().hasLabel('category').limit(5).valueMap()
```

统计个数使用 count():

```
gremlin> g.V().hasLabel('Disease').count()
```

### 查询某键值

```
gremlin> g.V().hasLabel('category').values('name')
```

### 查询节点某 label 的数量

```
gremlin> g.V().hasLabel('category').count()
```

### 查询多个键值

```
gremlin> g.V().hasLabel('category').values('name', 'description')
```

### 显示查询结果的长度

Select calculated column

```
gremlin> g.V().hasLabel('category').values('name').map{ it.get().length()}
```

### Distict

```
gremlin> g.V().hasLabel('category').values('name').map {it.get().length()}.dedup()
```

### 最大最小值

```
gremlin> g.V().hasLabel('category').values('name').map {it.get().length()}.max()
```

## Filtering (过滤)

### 相同

SQL:

```sql
SELECT ProductName, UnitsInStock FROM Products WHERE UnitsInStock = 0
```

Gremlin:

```
gremlin> g.V().has('product', 'unitsInStock', 0).valueMap('name', 'unitsInStock')
```

### 不同

SQL:

```sql
SELECT ProductName, UnitsOnOrder FROM Products WHERE NOT(UnitsOnOrder = 0)
```

Gremlin:

```
gremlin> g.V().has('product', 'unitsOnOrder', neq(0)).valueMap('name', 'unitsOnOrder')
```

## 指定范围

SQL:

```sql
SELECT ProductName, UnitPrice FROM Products WHERE UnitPrice >= 5 AND UnitPrice < 10
```

```
gremlin> g.V().has('product', 'unitPrice', between(5f, 10f)).valueMap('name', 'unitPrice')
```

### 多个过滤条件

Mulltiple filter conditions

SQL:

```sql
SELECT ProductName, UnitsInStock FROM Products WHERE Discontinued = 1 AND UnitsInStock <> 0
```

Gremlin:

```
gremlin> g.V().has('product', 'discontinued', true).has('unitsInStock', neq(0)).ValueMap('name', 'unitsInStock')
```

## 排序 Ordering

SQL:

```sql
SELECT ProductName, UnitPrice FROM Products ORDER BY UnitPrice ASC
SELECT ProductName, UnitPrice FROM Products ORDER BY UnitPrice DESC
```

Gremlin:

```
gremlin> g.V().hasLabel('product').order().by('unitPrice', incr).valueMap('name', 'unitPrice')
gremlin> g.V().hasLabel('product').order().by('unitPrice', decr).valueMap('name', 'unitPrice')
```

## 分页

```
gremlin> g.V().hasLabel("product').order().by('unitPrice', incr).range(5, 10).valueMap('name', 'UnitPrice')
```

参考：

> http://sql2gremlin.com

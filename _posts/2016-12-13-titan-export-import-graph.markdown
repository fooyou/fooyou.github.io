---
layout: post
title: Titan 导出 Graph 并在 Neo4j 展示
category: Document
tags: graph
date: 2016-12-13 18:12:48
published: true
summary: titan 没有提供可视化的图浏览工具，只能导出到 neo4j 等工具中查看
image: pirates.svg
comment: true 
---

Titan Graph 导出，Java 代码导出成 graphml(当然也能导出成 csv 等其他格式)：

```java
graph.io(IoCore.graphml()).writeGraph("graphml.xml");
```

Neo4j 导入

```sh
$ ./neo4j/bin/neo4j start
$ ./neo4j/bin/neo4j-shell -host 127.0.0.1 # 如果 /etc/hosts 中没有指定 127.0.0.1 为 localhost
neo4j-sh> import-graphml -i graphml.xml -t # -t 会加载 node 的 label，如果节点没有 labels 属性，需要修改为 labels
```

如果 labels 还是没有显示出来，可以在 neo4j 的浏览器界面中用语句修改：

```
MATCH (n)
WHERE n.label='PAGE'
SET n :PAGE
```

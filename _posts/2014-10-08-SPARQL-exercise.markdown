---
layout: post
title: SPARQL 练习
category: Document
tags: Semantic-Web SPARQL
year: 2014
month: 10
day: 08
published: true
summary: 看完《理解SPARQL》做了如下练习，使用dbpedia在线SPARQL进行，网址：http://dbpedia.org/snorql/
image: pirates.svg
comment: true
---

# SPARQL 练习

*看完《理解SPARQL》做了如下练习，使用dbpedia在线SPARQL进行，网址：http://dbpedia.org/snorql/*

练习前要先了解dbpedia的本体、属性、资源等

名称|URI|描述
---|---|---
本体|<http://dbpedia.org/ontology/>|[本体类](http://mappings.dbpedia.org/server/ontology/classes/)
属性|<http://dbpedia.org/property/>|描述一个本体属性
资源|<http://dbpedia.org/resource/>|任何东西都是资源

要查找一个linked data，必须首先知道其本体相关类和其属性定义，否则就无从查起。

## 查找中国所有的省份

在dbpedia中搜索河南（http://dbpedia.org/page/Henan）能看到对河南省的其中一个属性是：<http://dbpedia.org/class/yago/ProvincesOfThePeople'sRepublicOfChina>，那么利用这个属性我们可以找到所有的省份：

    PREFIX dbp_yago: <<http://dbpedia.org/class/yago/>
    
    SELECT ?p ?name WHERE {
        ?p a dbp_yago:ProvincesOfThePeople'sRepublicOfChina
        ?p rdfs:label ?name FILTER (LANG(?name) = 'zh')
    }

## 1.找出中国所有城市

首先我们要找的内容是城市，可以在DBPedia的本体库中找到类：City，而China已经是dbpedia resource中的结点了。

    PREFIX : <http://dbpedia.org/resource/>
    PREFIX dbpo: <http://dbpedia.org/ontology/>

    SELECT ?city, ?city_zh_name { 
        ?city a dbpo:City; dbpo:country :China .
        ?city rdfs:label ?city_zh_name FILTER (LANG(?city_zh_name) = 'zh') .
    }
    ORDER BY ?city

第1行对resource的PREFIX，没有命名，那么查询默认空间就是resource，第2行定义了dbpo为DBPedia的本体服务，select语句定义了两个变量`city`和`city_zh_name`。

`city`是类型为dbpedia-owl(<http://dbpedia.org/ontology/City>)属性dbpedia-owl:country所属国家为<http://dbpedia.org/resource/China>(:China)的城市，

`city_zh_name`意义为城市的中文名字，它是`city`的rdfs:label属性中的LANG()为'zh'的部分。

结果如下：

city | city_zh_name
---|---
[:%C3%9Cr%C3%BCmqi](http://dbpedia.org/snorql/?describe=http%3A//dbpedia.org/resource/%25C3%259Cr%25C3%25BCmqi) | "乌鲁木齐市"@zh
[:Alashankou](http://dbpedia.org/snorql/?describe=http%3A//dbpedia.org/resource/Alashankou) | "阿拉山口市"@zh
[:Altay_City]() | "阿勒泰市"@zh
[:Anda,_Heilongjiang](http://dbpedia.org/snorql/?describe=http%3A//dbpedia.org/resource/Altay_City) | "安达市"@zh
[:Anning,_Yunnan](http://dbpedia.org/snorql/?describe=http%3A//dbpedia.org/resource/Anning%2C_Yunnan) | "安宁市"@zh
... | ...

## 2.查找河南省所有的城市

    PREFIX : <http://dbpedia.org/resource/>
    PREFIX dbpo: <http://dbpedia.org/ontology/>

    SELECT ?city, ?city_zh_name { 
        ?city a dbpo:City; dbpo:isPartOf :Henan .
        ?city rdfs:label ?city_zh_name FILTER (LANG(?city_zh_name) = 'zh') .
    }
    ORDER BY ?city

查看dbpedia的owl:class:City，可以发现其属性有一个叫dbpedia-owl:isPartOf，

## 1. 查找所有在濮阳出生的人

<未完待续>

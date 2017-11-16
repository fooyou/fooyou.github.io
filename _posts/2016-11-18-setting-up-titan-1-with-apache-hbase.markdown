---
layout: post
title: 使用 Aapache Hbase 设置 Titan 1.0
category: Document
tags: bigdata
date: 2016-11-18 13:11:41
published: true
summary: 最近在研究图数据库，而 Titan 当前被认为是图数据库的引领者，Titan内部集成了 Aapache TinkerPop 的图协议栈，所以计划测试一下使用 Hadoop Hbase 做为存贮后端的 Titan 1.0。
image: pirates.svg
comment: true 
---

尽管 [Titan 官方文档](http://s3.thinkaurelius.com/docs/titan/1.0.0/) 已相当丰富并且有大量好的资源帮助你了解它是如何工作的，但这些示例相当普通，在如何选择和配置后端数据库上并没有提供一个完整清晰的配置和操作过程，而只提供了一些片段。（尤其对于HBase 的小白来说）。

在花了大量时间研究了大量的博客、论坛和讨论组后，终于把基于 HBase 数据库的 Titan 1.0 跑通了。所以本文整理了整个过程并希望可以帮助到其他研究 Titan 的小伙伴。

本文所涉及的内容：

- 安装 Hadoop
- 安装 HBase
- 安装 Titan
- 安装 Elasticsearch
- 运行 Gremlin 控制台并创建图
- 运行 Gremlin 服务以便能远程访问

**前期准备**

操作系统：MacOS Seirra

1. Java 8 环境，$JAVA_HOME 配置在环境变量中。
2. ssh 免密登陆
    ```
    $ ssh-keygen -t dsa -P '' -f ~/.ssh/authorized_keys
    $ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
    $ ssh localhost // 登陆测试
    ```

*因为是测试，所以 Hadoop 和 HBase采用的是伪分布配置*

## 安装配置 Hadoop
 
从官网下载 Hadoop 的二进制文件包，因为项目原因，我下载的是 hadoop-1.2.1-bin.tar，并解压到 /opt/hadoop-1.2.1 下。

*注意下载 HBase 包的时候里面的 Hadoop 版本要和这里的一致，否则可能会出现版本兼容引起的各种问题*

进入到 /opt/hadoop-1.2.1 下，修改以下配置文件：

1. conf/core-site.xml

    ```
    <configuration>
        <!-- Hadoop 伪分布式配置 -->
        <!-- 使用 /opt/hadoop-1.2.1/tmp 做为 hdfs 的存储目录，默认为 /tmp 
        <property>
            <name>hadoop.tmp.dir</name>
            <value>file:///opt/hadoop-1.2.1/tmp</value>
            <description>Abase for other temporary directories.</description>
        </property>
        -->
        <property>
            <name>fs.default.name</name>
            <value>hdfs://localhost:9000/</value>
        </property>
    </configuration>
    ```

2. conf/hdfs-site.xml

    ```
    <configuration>
        <property>
            <name>dfs.replication</name>
            <!-- 伪分布设置为 1 -->
            <value>1</value>
        </property>
        <!--
        <property>
            <name>dfs.namenode.name.dir</name>
            <value>file:///opt/hadoop/tmp/dfs/name</value>
        </property>
        <property>
            <name>dfs.datanode.data.dir</name>
            <value>file:///opt/hadoop/tmp/dfs/data</value>
        </property>
        -->
    </configuration>
    ```

3. conf/mapred-site.xml

    ```
    <configuration>
        <property>
            <name>mapred.job.tracker</name>
            <value>localhost:9001</value>
        </property>
    </configuration>
    ```

4. conf/Hadoop-env.sh

    将 JAVA_HOME 改为 JDK 目录

    ```
    export JAVA_HOME=`/usr/libexec/java_home`
    ```


配置完成，进入 Hadoop-1.2.1 主目录：

```
$ ./bin/Hadoop namenode -format // 格式化 Hadoop namenode
$ ./bin/start-all.sh            // 启动 Hadoop 的各个监护进程，可通过 http://localhost:50070 和 http://localhost:50030 查看 namenode 和 jobtracker。
$ jps                           // 查看 JVM 运行情况
$ ./bin/stop-all.sh             // 关闭 Hadoop 的各个监护进程
```

## 安装配置 HBase

从官方下载 hbase-0.98.23-hadoop1-bin.tar.gz （注意 titan hbase 中的 Hadoop 要和 Hadoop 版本一致），将其解压到 /opt/hbase-0.98.23-hadoop1 下，进入该目录，并完成以下配置：

1. conf/hbase-site.xml

    ```
    <configuration>
        <property>
            <name>hbase.rootdir</name>
            <!-- 端口号和ip地址要与 hadoop 配置参数保持一致 -->
            <value>hdfs://localhost:9000/hbase</value>
        </property>
        <property>
            <name>hbase.cluster.distributed</name>
            <value>true</value>
        </property>
    </configuration>
    ```

2. conf/hbase-env.sh

    ```
    export JAVA_HOME=`/usr/libexec/java_home` # 自己的JAVA_HOME 主目录
    export HBASE_CLASSPATH=/opt/hadoop-1.2.1/conf # 自己的 HADOOP_HOME 主目录
    ```

    如果要使用 HBase 自带的 Zookeeper 则需要 `HBASE_MANAGES_ZK=true`，这时启动和关闭顺序为：

    ```
    启动 Hadoop -> 启动 Zookeeper 集群 -> 启动 HBase -> 停止 HBase -> 停止 Zookeeper 集群 -> 停止 Hadoop。
    ```
完成以上步骤，Hadoop + HBase 伪分布式配置就完事了。

## 安装配置 Titan
 
从[官网下载](https://github.com/thinkaurelius/titan/wiki/Downloads)，当前版本是 titan-1.0.0-hadoop1.zip，要注意的是该包中的 Hadoop 版本要和 HBase 使用的 Hadoop 版本一致。
 
注意： Hadoop 的版本要一致 

## 安装 Elasticsearch

查看 titan-1.0.0-hadoop1.zip 包里的 Elasticsearch 的版本号为 1.5.1，那么需要安装 Elasticsearch-1.5.1 版本。

因为在 Mac 上，使用 homebrew 安装，但发现里面仅有 1.7 版本，所以就安装了 1.7，目前没发现什么问题，单最好是安装相同版本的。

```
$ brew search elasticsearch
elasticsearch homebrew/versions/elasticsearch17 homebrew/versions/elasticsearch24
$ brew install elasticsearch17
$ elasticsearch         # 运行 elasticsearch
(linux 上使用 service elasticsearch start)
```

当然 elasticsearch 配置文件中有许多配置项，现在全使用默认的值（/usr/local/Cellar/elasticsearch17/1.7.5/config/elasticsearch.yml，linux 上的位置在 /etc/elasticsearch/Elasticsearch.yaml）

## 运行 Gremlin 控制台

好了，前戏结束，现在到了启动各种服务，并运行 gremlin.sh 了。

启动顺序：

```
$ ./hadoop-1.2.1/bin/start-all.sh
$ ./hbase-0.98.23-hadoop1/bin/start-hbase.sh
$ elasticsearch
$ ./titan-1.0.0-hadoop1/bin/gremlin.sh
```

创建基于 hbase 和 elasticsearch 的图，并加载 titan 的测试数据：

```
gremlin> graph = TitanFactory.open('conf/titan-hbase-es.properties')
==>standardtitangraph[cassandrathrift:[127.0.0.1]]
gremlin> GraphOfTheGodsFactory.load(graph)
==>null
gremlin> g = graph.traversal()
==>graphtraversalsource[standardtitangraph[cassandrathrift:[127.0.0.1]], standard]
```

**全局图索引**

```
gremlin> saturn = g.V().has('name', 'saturn').next()
==>v[256]
gremlin> g.V(saturn).valueMap()
==>[name:[saturn], age:[10000]]
gremlin> g.V(saturn).in('father').in('father').values('name')
==>hercules
```

停止顺序：

```
$ (Ctrl + C 停止 gremlin)
$ (Ctrl + C 停止 elasticsearch)
$ /opt/hbase-0.98.23-hadoop1/bin/stop-hbase.sh  # 时间可能较长
$ /opt/hadoop-1.2.1/bin/stop-all.sh
```

## 参考：
 
> http://s3.thinkaurelius.com/docs/titan/1.0.0/getting-started.html
> https://www.revulytics.com/blog/setting-up-titan-1-0-apache-hbase



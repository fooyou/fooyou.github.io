---
layout: post
title: Solr 5.3的使用
category: Document
tags: solr
date: 2015-10-22 10:51:10
published: true
summary: 以前玩过4.9，转眼间都到5.3了，后悔以前没有做笔记，现在记录一下吧。
image: pirates.svg
comment: true
latex: false
---

以前用4.x的时候，用tomcat相当方便，官方有solr.war包，直接放到webapps下，运行tomcat搞定。现在一看solr5.3里没有了war包，不知道为什么，只有自己动手了。

**软件准备：**

- 下载 solr 5.3 版本：http://www.apache.org/dyn/closer.lua/lucene/solr/5.3.0
- 下载 tomcat：http://tomcat.apache.org/

## 运行环境配置

Tomcat根目录：/opt/apache/tomcat = TC
Solr Home目录：~/Workspace/solr_home = SH
Solr包解压位置：~/Downloads/solr-5.3.1 = SR

**步骤：**

1. 解压tomcat到/opt/apache/tomcat（路径自选）
2. 将 solr 压缩包中 /solr-5.3.1/server/solr-webapp/文件夹下有个webapp文件夹，将之复制到Tomcat/webapps目录下，并重命名成solr（随意，此名称在浏览器寻址时要用到）
3. 将 solr 压缩包中 solr-5.3.1/server/lib/ext 中的 jar 全部复制到 Tomcat/webapps/solr/WEB-INF/lib 目录中
4. 将 solr 压缩包中 solr-5.3.1/server/resources/log4j.properties 复制到Tomcat/webapps/solr/WEB-INF/lib 目录中
5. 将 solr 压缩包中 solr-5.3.1/server/solr 目录复制到计算机某个目录下，如~/solr_home
6. 打开Tomcat/webapps/solr/WEB-INF下的web.xml，找到如下配置内容（初始状态下该内容是被注释掉的）：

    ```
    <env-entry>
        <env-entry-name>solr/home</env-entry-name>
        <env-entry-value>/put/your/solr/home/here</env-entry-value>
        <env-entry-type>java.lang.String</env-entry-type>
    </env-entry>
    ```
    将<env-entry-value>中的内容改成你的solr_home路径，这里是~/solr_home

7. 保存关闭，而后启动tomcat，在浏览器输入http://localhost:8080/solr即可出现Solr的管理界面

## 创建Core并简单配置

Core Admin里的创建Core是不好用的，必须先创建solrconfig.xml和schema.xml后，方能创建Core，所以最简单的方法是Copy/Paste一个已经存在的Core或Collection，然后修改一些东西，存放在solr_home里，重启solr即可。

现在我们开始导入solr包里自带的example/films，先来体验一下：

**步骤：**

1. 创建Core films

    cd到solr解压包位置，cd SR

    ```
    $ bin/solr start                    # 启动solr
    $ bin/solr create -c films          # 创建Core films
    ```

    这种方式启动的Solr home是 SR/server/solr，刚创建的films就在这里，把它Copy到tomcat配置的solr home下SH里，重启tomcat就可以看到它了。接下来的films操作就以tomcat的那个为准了。

2. 添加两个字段的schema：

    ```
    $ curl http://localhost:8080/solr/films/schema -X POST -H 'Content-type:application/json' --data-binary '{
        "add-field": {
            "name": "name",
            "type": "text_general",
            "stored": true
        },

        "add-field": {
            "name": "initial_release_date",
            "type": "tdate",
            "stored": true
            }
    }'
    ```

---
layout: post
title: mallet build JDK 1.7版本的jar包
category: Document
tags: java ant
date: 2015-11-10 11:03:10
published: true
summary: mallet源码包使用JDK1.8 build JDK 1.7兼容版本
image: pirates.svg
comment: true
latex: false
---

官方下载的mallet jar包javac编译器使用的是52,也就是JDK 1.8，但是App需要1.7的，而我们开发机上装的全是1.8，所以需要编译mallet。

尝试步骤如下：

## 首先尝试：修改pom.xml:

把maven编译器的source和target设置为1.7，然后执行命令：`ant jar`

```
  <properties>
    <maven.compiler.source>1.7</maven.compiler.source>
    <maven.compiler.target>1.7</maven.compiler.target>
    <encoding>UTF-8</encoding>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>
```

编译成功，但目标失败，使用`file *.class` 发现 Version 还是 52.0，没有将为51

## 再次尝试：修改build.xml java_version:

第12行修改为1.7

```
  <property name="java_version" value="1.7"/>
```

编译成功，但目标失败，使用`file *.class` 发现 Version 还是 52.0，没有将为51

## 最后尝试：修改build.xml javac:

第53行开始，指定source和target的版本号，不使用默认的：

```
    <javac
      source="1.7"
      target="1.7"
      destdir="${class}"
      classpathref="project.classpath"
      debug="true"
      deprecation="off"
      listfiles="no"
      >
      <src path="${src}"/>
      <include name="cc/**/*.java"/>
	  <!-- compilerarg value="-Xlint:unchecked"/ --> 
    </javac>
```

编译成功，但目标成功。

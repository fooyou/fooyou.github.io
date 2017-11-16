---
layout: post
title: Maven如何把本地依赖的jar包打包进无依赖的可执行的jar包
category: Document
tags: maven
date: 2016-01-15 18:01:25
published: true
summary: 原型、验证，总是需要把当前的工程做成一个无依赖的可执行jar包以方便测试，maven库的依赖使用 maven-assembly-plugin jar-with-dependences 就可以了，可是对于本地依赖怎么打包唻？
image: pirates.svg
comment: true
---

本文只记录如何做？至于为什么没有探究。

## Step 1：pom.xml 中引入本地jar包

_最好是把本地jar包安装到本地maven仓库里去_

添加dependency，设置其scope为system，然后指定依赖的jar包路径，如下：

```xml
<dependency>
    <groupId>cc.mallet</groupId>
    <artifactId>mallet</artifactId>
    <version>2.0.8RC2</version>
    <scope>system</scope>
    <systemPath>${project.basedir}/lib/mallet.jar</systemPath>
</dependency>
```


## Step 2：配置 maven-assembly-plugin

在 pom.xml 中添加如下配置

```xml
<plugin>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>2.2.1</version>
    <configuration>
        <descriptors>
            <descriptor>assembly.xml</descriptor>
        </descriptors>
        <!--descriptorRefs>         
            <descriptorRef>jar-with-dependencies</descriptorRef>       
        </descriptorRefs-->       
        <archive>         
            <manifest>           
                <mainClass>org.apache.lucene.Search</mainClass>         
            </manifest>       
            
            <manifestEntries>
                <PCD-Version>${GIT_COMMIT}</PCD-Version>
            </manifestEntries>
        </archive> 
    </configuration>
    
    <executions>
        <execution>
            <id>make-assembly</id>
            <phase>package</phase>
            <goals>
                <goal>single</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

## Step 3：编写 assembly.xml 文件

```xml
<assembly xmlns="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.0" 
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.0 http://maven.apache.org/xsd/assembly-1.1.0.xsd">
  <!-- TODO: a jarjar format would be better -->
  <id>full-${GIT_COMMIT}</id>
  <formats>
    <format>jar</format>
  </formats>
  <includeBaseDirectory>false</includeBaseDirectory>
  <dependencySets>
    <dependencySet>
      <outputDirectory>/</outputDirectory>
      <useProjectArtifact>true</useProjectArtifact>
      <unpack>true</unpack>
      <scope>runtime</scope>
    </dependencySet>
    <dependencySet>
      <outputDirectory>/</outputDirectory>
      <useProjectArtifact>true</useProjectArtifact>
      <unpack>true</unpack>
      <scope>system</scope>
    </dependencySet>
  </dependencySets>
</assembly>
```

然后，编译就行了：

```
$ maven assembly:assembly
```

成功！

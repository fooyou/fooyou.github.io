---
layout: post
title: elementary os 网络代理下 build thrift
category: Document
tags: linux thrift build
date: 2015-08-20 15:46:00
published: true
summary: maven setting代理设置过了，为嘛不好用，最终build成功了，为嘛 sudo make install 又不好用，如何解决？
image: pirates.svg
comment: true
latex: false
---

之前就在开发机上Build过，好像遇到什么代理的问题了，但很快解决了，没有留下什么记录。等过了好长时间又需要用到thrift，这时环境变了，一切都得重来，依稀记得设置的方式，却又遇到不同的问题。

我下载的版本是0.9.2的Release包，所以不用安装Boost。

安装Ubuntu需要的编译环境（是的elementary os来自ubuntu）：

```
sudo apt-get install libboost-dev libboost-test-dev libboost-program-options-dev libboost-system-dev libboost-filesystem-dev libevent-dev automake libtool flex bison pkg-config g++ libssl-dev
```

thrift会自动查找当前的开发语言环境，因为我只需用到python、java和C++并且环境都配好了，所以直接Configure：

```
$ ./configure
```

_代理配置（我没有用这种方式）_

```
$ ./configure --with-java ANT_FLAGS='-Dproxy.enabled=1 -Dproxy.host=proxy.xxx.com -Dproxy.port=8080'
```

没有问题，然后：

```
$ make
```

遇到问题了，执行到 `mvn.ant.tasks.download` 不动了，肯定是代理问题，按记忆找到了 ./thrift-0.9.2/lib/java/build.pom第292行的proxy设置，修改后make成功。

```xml
    <target name="proxy">
        <setproxy proxyhost="proxy.xxx.com" proxyport="8080" />
    </target>
```

该安装了：

```
sudo make install
```

本以为没问题了，可是就在`mvn.ant.tasks.download`后，有个下载mvn依赖的`mvn.init`停住不动了，肯定又是代理的事，可是在 ~/.m2/settings.xml里我已经配置了代理啊，一直再用啊～～

然后终于看到了`sudo`root用户啊，于是在root用户下创建了 ~/.m2/settings.xml，OK万事大吉！

settings.xml内容如下：

```xml
<settings>
    <proxies>
        <proxy>
            <active>true</active>
            <protocol>http</protocol>
            <host>proxy.xxx.com</host>
            <port>8080</port>
            <!--
            <username>proxyuser</username>
            <password>somepassword</password>
            <nonProxyHosts>www.google.com|*.somewhere.com</nonProxyHosts>
            -->
        </proxy>
    </proxies>
    <profiles>
        <profile>
            <id>jdk-1.8</id>
            <activation>
                <activeByDefault>true</activeByDefault>
                <jdk>1.8</jdk>
            </activation>
            <properties>
                <maven.compiler.source>1.8</maven.compiler.source>
                <maven.compiler.target>1.8</maven.compiler.target>
                <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
            </properties>
        </profile>
    </profiles>
</settings>
```

总之好用了。


> 参考：https://thrift.apache.org/docs/BuildingFromSource

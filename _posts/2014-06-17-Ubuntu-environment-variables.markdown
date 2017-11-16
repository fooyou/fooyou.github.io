---
layout: post
title: Ubuntu设置环境变量并立即生效
category: Document
tags: image
year: 2014
month: 06
day: 17
published: true
summary: Ubuntu系统修改系统环境变量的最好方法是在/etc/profile.d/文件夹下编写sh文件，而非直接修改/etc/profile文件。
image: pirates.svg
comment: true
---

Linux系统包含两类持久性环境变量：用户环境变量(Session-wide)和系统环境变量(System-wide)。用户环境变量仅仅对当前的用户有效，系统环境变量对所有用户都有效。

## 用户环境变量

用户环境变量通常被存储在下面的文件中：

- ~/.profile
- ~/.pam_environment
- ~/.bash_profile 或者 ~/.bash_login
- ~/.bashrc

## 系统环境变量

- /etc/environment
    - **注意：**这不是个脚本文件，其不支持变量扩展，也不需要使用赋值表达式（无需使用export）

    ```
    JAVA_HOME=/opt/java/jdk1.8.0_20
    ```
- /etc/profile
    - 这个文件通常被认为是设置系统级环境变量的文件，但是这是基础文件包的配置文件，不建议直接修改这个文件，而是在/etc/profile.d目录下添加自定义配置文件.
- /etc/profile.d/*.sh
    - 查看/etc/profile的代码可以发现，其中有对/etc/profile.d/*.sh读取并设置的代码。


------

所以比如要配置JAVA的环境变量，那么ubuntu上最好的做法应该如下：

1. 在/etc/profile.d下新建文件java_var.sh

    ```
    vi /etc/profile.d/java_var.sh
    ```

2. 加入JAVA_HOME、JRE_HOME和PATH到java_var.sh

    ```
    export JAVA_HOME=/opt/java/jdk1.8.0_20
    export JRE_HOME=$JAVA_HOME/jre
    export CLASSPATH=$CLASSPATH:.:$JAVA_HOME/lib:$JRE_HOME/lib
    export PATH=$PATH:$JAVA_HOME/bin
    ```

3. 使配置生效

    ```
    source /etc/profile
    ```

4. 查看是否设置成功。（使用echo或printenv命令）

    ```
    echo $JAVA_HOME
    printenv JAVA_HOME
    ```

这样设置，各个环境变量之间，只需维护自己的文件，相互之间的影响较低，方便维护。


### 参考资料

> https://help.ubuntu.com/community/EnvironmentVariables#Persistent_environment_variables
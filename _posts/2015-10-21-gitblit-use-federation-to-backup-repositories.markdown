---
layout: post
title: 使用federation来备份gitblit中的仓库
category: Document
tags: git
date: 2015-10-21 09:13:58
published: true
summary: So, 不管代码托管有多安全，还是要备份的
image: pirates.svg
comment: true
latex: false
---

## Federation Client

Gitblit提供的gitblit instance之间联携备份的客户端工具，可干的活挺多，目前仅用于对gitblit的仓库进行备份。

下面简要介绍下配置过程：

### 软件准备

1. [gitblit](http://gitblit.com) war包，当然也可以用go gitblit或jar包，但为了方便tomcat使用，我们用的是jar包。
2. fedclient 客户端
3. tomcat（可选）

### 配置过程

__gitblit server端__

1. 将gitblit.war放在./tomcat/webapps下，启动tomcat，./tomcat/bin/startup.sh。
2. 修改文件./tomcat/webapps/gitblit/WEB-INF/data/gitblit.properties中的`federation.passphrase`，随便填几个和Java不冲突的字符串或者数字等>即可（该选项上面有完整的说明）。
3. 重启tomcat
4. 在tomcat/logs/catalina.out中搜索`token`，就会找到类似如下信息（官方说web ui里用admin也能看到，不过没找到地方）

    ```
    2015-10-21 13:35:19 [INFO ] Federation ALL token = 6d8s9ab981a0c9ccaaa5373f68af20f00222a938
    2015-10-21 13:35:19 [INFO ] Federation USERS_AND_REPOSITORIES token = 7xc92d8aad61f204dc2517057405ece157a5c1df6
    2015-10-21 13:35:19 [INFO ] Federation REPOSITORIES token = cffea3979020a588d9w0b7c3ede9bf6c8b17d522
    ```

__备份server端（使用fedclient）__

1. 把fedclient-*.zip解压到某个地方，假定为~/Workspace/gitblit_backup.
2. 修改文件~/Workspace/gitblit_backup/federation.properties中的配置项，最重要的是把上述token和url填上。（这里我们用的是ALL）
3. 使用crontab设置定时任务定时执行脚本进行备份。


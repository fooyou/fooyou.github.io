---
layout: post
title: 网络代理环境下使用ProxyCommand ssh 连接 github 和 bitbucket
category: Document
tags: git ssh
date: 2016-07-07 10:07:17
published: true
summary: 之前有篇文章介绍代理下 ssh 连接 github，写了个脚本。这篇介绍使用 ProxyCommand 命令 corkscrew 连接
image: pirates.svg
comment: true
---

## 一、安装 corkscrew

```
$ sudo apt-get install corkscrew
```

## 二、配置 ssh

```
# Format:
#   host <alias>
#       hostname <FQDN or IP access>
#       user <remote user>
#       port <remote port>
#       IdentityFile <local path to private key file>

ProxyCommand /usr/bin/corkscrew <ip_address> <port> %h %p

# Github access
Host github.com
    Hostname ssh.github.com
    Port 443

# Bitbucket.orgs
host bitbucket.org
    hostname altssh.bitbucket.org
    Port 443
```

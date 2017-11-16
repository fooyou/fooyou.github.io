---
layout: post
title: CentOS iptables如何能打开端口
category: Document
tags: port linux iptables
date: 2015-09-01 15:42:27
published: true
summary: iptables打开端口为什么不好用，需要disable selinux，这又是嘛东东？
image: pirates.svg
comment: true
latex: false
---

之前搞过，好用了，重装服务器后，又不好用了，重搞发现还没好用，好记性不如烂笔头啊！

关键是对文件的 /etc/systemctl/iptables的修改，未修改的CentOS 7的文件如下：

```
# sample configuration for iptables service
# you can edit this manually or use system-config-firewall
# please do not ask us to add additional ports/services to this default configuration
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
COMMIT
```

然后开始在COMMIT前，添加参考 22 端口的语句：

```
-A INPUT -p tcp -m state --state NEW -m tcp --dport 8081 -j ACCEPT
```

重启服务：

```
$ service iptables restart
```

查看修改：

```
$ iptables -L -n
```

可以看到8081端口已经打开，可是却无法访问，折腾半天，突然发现其上这两句：

```
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
```

难道先执行这两句，再执行打开8081的端口就有问题，一试果然如此，调整一下顺序就OK了，最终如下：

```
# sample configuration for iptables service
# you can edit this manually or use system-config-firewall
# please do not ask us to add additional ports/services to this default configuration
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 8081 -j ACCEPT
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
COMMIT
```

iptables，以后有时间再慢慢研究吧！


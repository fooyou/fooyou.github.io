---
layout: post
title: elementaryos 0.4(Ubuntu 16.04) 安装搜狗输入法
category: Document
tags: linux
date: 2016-10-11 17:10:02
published: true
summary: 我的翻墙选择
image: pirates.svg
comment: true
---

先是下载 deb 文件，安装一大推依赖没解决，烦！

然后使用 gdebi 安装（自动安装依赖），正高兴呢，处错误了


```
# gdebi sogoupinyin_2.0.0.0078_amd64.deb 
Reading package lists... Done
Building dependency tree        
Reading state information... Done
Reading state information... Done

Sogou Pinyin Input Method
 Based on web search engine technology, Sogou Pinyin input method is
 the next-generation input method designed for Internet users. As it
 is backed with search engine technology, user input method can be
 extremely fast, and it is much more advanced than other input method
 engines in terms of the volume of the vocabulary database and its
 accuracy. Sogou input method is the most popular input methods in
 China, and Sogou promises it will always be free of charge.
Do you want to install the software package? [y/N]:y
(Reading database ... 151277 files and directories currently installed.)
Preparing to unpack sogoupinyin_2.0.0.0078_amd64.deb ...
Unpacking sogoupinyin (2.0.0.0078) over (2.0.0.0078) ...
Setting up sogoupinyin (2.0.0.0078) ...
Processing triggers for mime-support (3.59ubuntu1) ...
Processing triggers for bamfdaemon (0.5.3~bzr0+16.04.20160523-0ubuntu1) ...
Rebuilding /usr/share/applications/bamf-2.index...
Processing triggers for gnome-menus (3.13.3-6ubuntu3.1) ...
Processing triggers for desktop-file-utils (0.22-1ubuntu5+elementary2~ubuntu0.4.1) ...
Processing triggers for shared-mime-info (1.5-2ubuntu0.1) ...
Processing triggers for hicolor-icon-theme (0.15-0ubuntu1) ...
Processing triggers for libglib2.0-0:amd64 (2.48.1-1~ubuntu16.04.1) ...
No such key 'Gtk/IMModule' in schema 'org.gnome.settings-daemon.plugins.xsettings' as specified in override file '/usr/share/glib-2.0/schemas/50_sogoupinyin.gschema.override'; ignoring override for this key.
```

查看一下 /usr/share/glib-2.0/schemas/50_sogoupinyin.gschema.override. 据说应该是：

```
[org.gnome.settings-daemon.plugins.keyboard]
active=false
[org.gnome.settings-daemon.plugins.xsettings]
overrides={'Gtk/IMModule':<'fcitx'>}
[com.canonical.indicator.keyboard]
visible=false
```

而这个包里的却是：

```
[org.gnome.settings-daemon.plugins.keyboard]
active=false
[org.gnome.settings-daemon.plugins.xsettings]
Gtk/IMModule=fcitx
[com.canonical.indicator.keyboard]
visible=false
```

但是，直接忽略掉它，找到 Language Setting 点击添加"+" 号，将 sogou 加入，就好用了。

> https://bbs.archlinux.org/viewtopic.php?id=206234

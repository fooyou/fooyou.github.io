---
layout: post
title: 在elementary os中修改终端字体
category: Document
tags: elementaryos
date: 2015-12-07 14:12:52
published: true
summary: 看手机玩Pad眼睛受伤啊，这不又近视了，必须调大终端字体才能看清楚了。
image: pirates.svg
comment: true
---

## 方法一：安装 dconf-editor

```
$ sudo apt-get install dconf-tools
```

安装完成后打开App， 找到 org-> gnome -> desktop -> interface -> monospace-font-name 双击后按下面的语法输入 \[font-name\]\[property\]\[size\]，比如：`Droid Sans Mono 12`。

这里设置的是系统通用等宽字体的地方，终端等地方的字体默认就取自这里。

还可以只针对终端设置字体类型：org -> pantheon -> terminal -> settings -> font 输入要设置的值语法同上。


## 方法二：使用 gsettings 命令设置

```
$ gsettings set org.gnome.desktop.interface monospace-font-name 'Droid Sans Mono 12'
```

或者：

```
$ gsettings set org.pantheon.terminal.settings font 'Droid Sans Mono 12'
```

当然其他的选项也可以通过这种方式设置。

PS:

## 屏蔽 pantheon terminal 中的 Ctrl C/V 的快捷键

对使用 vim 的开发者来说，EOS 的终端内置了 Ctrl C/V 的拷贝和粘贴，这样使用 vim 就不能多选了，所以想屏蔽掉使用 ubuntu 中默认的 Ctrl Shift C/V 进行相关操作，使用 gsettings 查看下有无相关设置项：

```
$ gsettings list-schemas | grep terminal
org.pantheon.terminal.settings
org.gnome.desktop.default-applications.terminal
org.pantheon.scratch.plugins.terminal
org.pantheon.terminal.saved-state 
```

查看下org.pantheon.terminal.settings 下有无 copy paste 选项：

```
$ gsettings list-keys org.pantheon.terminal.settings
encoding
foreground
palette
unsafe-paste-alert
tab-bar-behavior
cursor-shape
background
natural-copy-paste
font
allow-bold
remember-tabs
alt-changes-tab
cursor-color
save-exited-tabs
follow-last-tab
audible-bell
shell
scrollback-lines
```

可以看到有个值叫 natural-copy-paste，貌似很像，值是真没呢？

```
$ gsettings get org.pantheon.terminal.settings natural-copy-paste
true
```

把它设置为 false 后看看效果

```
$ gsettings set org.pantheon.terminal.settings natural-copy-paste false
```

关闭重新打开 terminal，点开 vim 发现功能和 ubuntu 一样了，哈哈！

参考：

> http://elementaryos.stackexchange.com/questions/1149/how-can-i-chage-the-default-font/1153#1153


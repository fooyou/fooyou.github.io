---
layout: post
title: 使用 fbterm 在tty终端中显示和输入中文
category: Document
tags: linux
date: 2015-11-30 14:11:59
published: true
summary: 为什么要这么骨灰？一个是因为好奇，二是因为目前的工作环境需要。
image: pirates.svg
comment: true
---

前面说到了终端文本网页浏览器w3m，仅仅是觉得好玩，后来在桌面系统中使用Chrome浏览器加载flash的时候整个桌面系统会无缘无故的无应答，无应答后切换到tty，强制logout，restart桌面系统，全都不好用。而这时候，我可能正在使用vim在编写代码或者文档，如果重启电脑的话，可能打断正在做的事情（码农最讨厌的事）。

所以为了正在做的工作不会因为桌面系统的崩溃而遭到打断，我需要在tty下正常显示和输入中文，前文写过zhcon是不支持x64系统的，尽管可以修改代码重新编译但我很懒的，哥喜欢现成的。

所以找到了 fbterm 这个工具，装上后运行即可显示中文，但输入还不行，于是google了一番找到了这个文章 [fbterm](https://fcitx-im.org/wiki/Fbterm)


## Step 1: 安装fbterm fcitx-fbterm

```
$ sudo apt-get install fbterm fcitx fcitx-fbterm fcitx-chewing fcitx-configtool
```

## Step 2: 把普通用户加入video group

```
$ sudo gpasswd -a <your username> video
```

或者是这样（但不推荐因为这么做修改了 setuid）

```
$ sudo chmod u+s /usr/bin/fbterm
```

## Step 3: 让普通用户下的fbterm能接受系统按键（来进行输入法切换）

```
$ sudo setcap 'cap_sys_tty_config+ep' /usr/bin/fbterm
```

_注意：_ 如果你不是在使用KMS（Kernel mode settings，开源显示驱动），就必须使用root账户来运行fbterm。

## Step 4: 打开并编辑~/.fbtermrc文件

该文件可以设置字体，编码，输入法等，我们要把`input-method`设置成fcitx-fbterm，如下：

```
$ vim ~/.fbtermrc
```

```
font-name=Ubuntu Mono
font-size=18
input-method=fcitx-fbterm
```

关于中文显示的非utf-8编码问题，可在运行fbterm的时候添加LANG的方式实现，这里与fbterm无关，省略。

## Step 5： 检测运行状态

```
$ fcitx-fbterm-helper
```

如果显示器个数大于1，可通过 -d参数选择显示以能正常运行：

```
$ fcitx-fbterm-helper -d 1
```

如果运行遇到问题可以加 -l 参数 重新加载 fcitx：

```
$ fcitx-fbterm-helper -l
```

再有问题自求多福吧！

## 遇到的问题

1. Cannot communicate fcitx with Dbus...
    fcitx和fbterm使用顺序的问题，fbterm下使用fcitx-fbterm-helper，先加载fcitx后在调用fbterm就不会有问题了。
2. stdin isn't a interactive tty
    不影响正常使用，据说再跑一次就没问题了。

参考：

> https://fcitx-im.org/wiki/Fbterm

---
layout: post
title: Flask系列教程之一：Hello World
category: Document
tags: flask
year: 2015
month: 06
day: 11
published: true
summary: 跟随大师 Miguel Grinberg 的脚步学习Flask，系列教程一认识Flask。
image: pirates.svg
comment: true
---

*Miguel Grinberg是个很牛的软件工程师，尤其网络工程师，用过很多的语言编写过web app，包括PHP、Ruby、Smalltalk，甚至是C++，他认为Python/Flask相接合是其中最灵活易用的组合（我也这么认为，至少现在），他还编写了一本书叫做《Flask Web Development》都是在开发中总结的技术经验，2014年出版（[点我下载](http://cdn4.filepi.com/g/bDQKMSb/1434087411/4cdb09017d4e0bc7048aa7c0eb437cb4)）。*

*通过这系列教程，你可以使用Flask开发一个blog，然而学到的东西可远不止这些。既然有人(还是个大牛人)，分享了开发经验，那我们就站在他的肩膀上吧，开始～～*

## 关于教程中使用的App——_microblog_

microblog将逐渐涵盖以下内容：

- 用户管理，包括登录、会话、用户角色、profile和用户替身（user avatars）
- 数据库管理，包括迁移处理(migration handling)
- 网页form支持，包括值验证
- 分页
- 全文检索
- 邮件提醒
- HTML模板
- 多语言支持
- 缓存以及其他性能优化
- 开发以及产品服务器的调试技术
- 产品服务器的安装

可见，这不仅是一个blog，而是涵盖了Web App开发的所有流程，然后你就可以用flask开发其他Web App了。

## 系统需求

你有一台能运行Python的电脑，最好你习惯于命令行开发（terminal for linux and command prompt for Windows）和你的操作系统的基础的文件管理。

还有，你得懂Python！^_^

## 安装Flask

如果没安装Python，[安装](http://python.org/download/)

每个Python开发者都会强烈推荐你使用Python虚拟环境 [Virtual environment](http://pypi.python.org/pypi/virtualenv)来安装你的App环境，好处多多，我们也使用这个。

选一个位置，创建一个microblog的文件夹，我们就在这里干活了。

如果你使用Python3.4+，那么其已内置了vitual evn，直接在根目录下执行：

```
$ python -m venv flask
```

其他版本的Python的安装，如果在Mac上可以：

```
$ sudo easy_install virtualenv
```

如果在linux上，比如ubuntu，可以：

```
$ sudo apt-get install python-virtualenv
```

Windows用户最让人头痛，所以最好安装Python3.4，否则最好安装pip，然后在命令行，这样安装：

```
pip install virtualenv
```

等安装完毕后，在根目录下使用以下命令创建虚拟环境：

```
$ virtualenv flask
```

然后到flask/bin目录下，激活虚拟环境：

```
$ source flask/bin/activate
```

_注：关闭虚拟环境直接使用deactivate_

```
$ deactivate
```

开始安装：

Linux, OS X, 或者Cygwin,通过以下命令安装flask和其扩展，一个一个安装：

```
$ flask/bin/pip install flask
$ flask/bin/pip install flask-login
$ flask/bin/pip install flask-openid
$ flask/bin/pip install flask-mail
$ flask/bin/pip install flask-sqlalchemy
$ flask/bin/pip install sqlalchemy-migrate
$ flask/bin/pip install flask-whooshalchemy
$ flask/bin/pip install flask-wtf
$ flask/bin/pip install flask-babel
$ flask/bin/pip install guess_language
$ flask/bin/pip install flipflop
$ flask/bin/pip install coverage
```

Windows上，有点不同：

```
$ flask\Scripts\pip install flask
$ flask\Scripts\pip install flask-login
$ flask\Scripts\pip install flask-openid
$ flask\Scripts\pip install flask-mail
$ flask\Scripts\pip install flask-sqlalchemy
$ flask\Scripts\pip install sqlalchemy-migrate
$ flask\Scripts\pip install flask-whooshalchemy
$ flask\Scripts\pip install flask-wtf
$ flask\Scripts\pip install flask-babel
$ flask\Scripts\pip install guess_language
$ flask\Scripts\pip install flipflop
$ flask\Scripts\pip install coverage
```

这些包我们将在microblog中用到。


## "Hello, World"

> Refer: http://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world

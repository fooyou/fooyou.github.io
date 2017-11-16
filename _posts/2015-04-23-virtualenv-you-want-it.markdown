---
layout: post
title: virtualenv解决Python开发环境问题的神器
category: Document
tags: virtualenv python
year: 2015
month: 04
day: 23
published: true
summary: 当你拥有多个python项目且每个项目的Python环境都不尽相同时，virtualenv就是拯救你Python环境的神器。
image: pirates.svg
comment: true
---

**virtualenv**

------

*使用Python做一些简单的事已成了我现在的习惯，越来越喜欢Python这门语言和其强大到让人生畏的社区，正是有了活跃的社区我们在开发中可以方便的使用各种包以完成我们的项目，然而安装那么多的包到我们的服务器会带来很多管理上的麻烦，什么版本依赖啦，向后不兼容啦，有时让人烦不胜烦。然后神器来了，欢迎virtualenv。*

virtualenv是能创建各自孤立的Python环境的工具，它能为每个不同项目提供一份“独立”的Python环境。当然它并没有真正安装多个Python副本，却是以一种“巧妙”的方式来让各个项目保持独立。

virtualenv可为部署应用提供方便，把开发环境的虚拟环境打包生产环境即可，不需要在服务器上在搭建一遍。

### 安装

在Mac OS X或linux下，使用以下命令

```
$ sudo easy_install virtualenv
```

或更好的使用pip：

```
$ sudo pip install virtualenv
```
*使用 `-E`参数可以使用系统环境变量中的http-proxy*

ubuntu下也可以直接使用apt-get：

```
$ sudo apt-get install python-virtualenv
```

如果以前没有安装，那么默认会安装在/usr/local/bin下，使用pip3安装python3版本的virtualenv会加上版本号后缀，比如我现在用的是python3.4，所以我现在的有两个版本的virtualenv：

```
$ ls /usr/local/bin/ | grep virtualenv
virtualenv
virtualenv-3.4
```

要创建python3的虚拟环境，就要使用`vitualenv-3.4 [PRJ]`

### 创建环境

安装完毕后，打开terminal就可以创建自己的环境。通常先创建一个项目文件夹，然后使用virtualenv创建虚拟环境 ENV（当然名字自己选）

```
$ mkdir myproject
$ cd myproject
$ virutlaenv ENV
Using base prefix '/usr'
New python executable in project/bin/python3
Also creating executable in project/bin/python
Installing setuptools, pip...done.
```

我的是12.1.1版本，默认生成的环境是不带系统级 site-packages 的，但以前的什么版本默认是带的，不带比较好。可通过命令查看其他参数 `virtualenv --help`

查看ENV目录可发现：

```
├── bin
│   ├── activate
│   ├── activate.csh
│   ├── activate.fish
│   ├── activate_this.py
│   ├── easy_install
│   ├── easy_install-3.4
│   ├── pip
│   ├── pip3
│   ├── pip3.4
│   ├── python -> python3
│   ├── python3
│   └── python3.4 -> python3
├── include
│   └── python3.4m -> /usr/include/python3.4m
└── lib
    └── python3.4
```

### 启动虚拟环境

```
$ cd ENV
$ ./bin/activate 
```

OK，现在就已经激活了virtualenv，然后使用pip或者easy_install，或者直接代码安装第三方包了

```
pip install Flask
```

如果没有启动虚拟环境，系统也安装了pip，那么第三方包将被装在系统环境中，为避免此类事情发生，可以在环境变量中（~/.bashrc, etc）添加变量：

```
export PIP_REQUIRE_VIRTUALENV=true
```

### 退出虚拟环境

退出virtualenv：

```
$ ./bin/deactive
```

------

**virtualenvwrapper**

Virtualenvwrapper是virtualenv的扩展包，用于更方便的管理虚拟环境。

### 安装

```
$ sudo pip install virtualenvwrapper
```

此时尚不能使用，实际上需要运行`/usr/local/bin/virtualenvwrapper.sh`也能使其正常工作，打开这个文件里面有安装说明：

```
# Setup:
#
#  1. Create a directory to hold the virtual environments.
#     (mkdir $HOME/.virtualenvs).
#  2. Add a line like "export WORKON_HOME=$HOME/.virtualenvs"
#     to your .bashrc.
#  3. Add a line like "source /path/to/this/file/virtualenvwrapper.sh"
#     to your .bashrc.
#  4. Run: source ~/.bashrc
#  5. Run: workon
#  6. A list of environments, empty, is printed.
#  7. Run: mkvirtualenv temp
#  8. Run: workon
#  9. This time, the "temp" environment is included.
# 10. Run: workon temp
# 11. The virtual environment is activated.
#
```

使用方法命令：

   Command   |   Description
-------------|-----------------------
workon       | 列出虚拟环境列表。workon [name] 启动/切换虚拟环境
lsvirtualenv | 同上
mkvirtualenv | 新建虚拟环境。mkvirtualenv [name]
rmvirtualenv | 删除虚拟环境
deactive     | 关闭虚拟环境

## 参考：

> https://virtualenv.pypa.io/en/latest/


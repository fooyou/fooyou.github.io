---
layout: post
title: python使用pdb调试
category: Document
tags: python debug
year: 2014
month: 05
day: 13
published: true
summary: 对于从Windows迁移来的开发者而言在Terminal里进行调试是很别扭的事，本文介绍如何使用pdb对python代码进行调试。
image: pirates.svg
comment: true
---

根据设计编写代码后需要对一些代码进行调试，可以通过log信息检查代码是否按照设计执行，有时发现代码工作异常但看log又不能提供更多信息时，就需要对代码进行调试以获得更详细的出错信息来帮助修正代码。

python提供了原生的pdb来对代码进行调试还是比较方便的，但像远程调试，多线程之类的pdb是搞不定的。

pdb调试方法有以下几种：

------

1. 命令行启动目标，加上`-m`参数，这是我最常用的调试方式。

	```
	python -m pdb test.py
	``` 

	执行后，断点会出现在程序的第一行

2. 在python的交互环境启用

	```python
	import pdb
	import mymodule
	pdb.run('mymodule.test()')
	```
3. 硬编码调试
	
	```python
	import pdb
	for i in range(10):
		if i == 5:
			pdb.set_trace()
		print(i)
	```
	运行脚本，到pdb.set_trace()那会中断。

常用调试命令：

------

   Key   | Description
---------|-------------
h(elp)   | 打印当前pdb可用命令。可用`h [command]`查询
l(ist)   | 列出当前运行的代码块
b(reak)  | b [n] 在第n行设置断点， b [func] 在某函数入口设置断点，只输入`b`会列出所有断点
cl(ear)  | cl [n] 清除第n行断点，cl [func], cl 同 b
enbale   | 激活断点
disable  | 禁用断点
n(ext)   | 单步出调试 Step out，相当于VS的F10
s(tep)   | 单步进调试 Step into，相当于VS的F11
c(ont(inue)) | 继续执行，直至遇到断点，相当于VS F5
j(ump)   | 跳转到某行 j [n]
a(rgs)   | 打印当前函数参数
p(arameter) | 打印变量
!        | 感叹号后跟语句，可直接改变某个变量
q(uit)   | 退出调试
condition| 条件断点 condition [n] 条件表达式
bt/w     | 查看调用堆栈
r        | 执行到函数返回
d        | 移到当前堆栈帧的下一帧
u        | 移到当前堆栈帧的上一帧

艾玛，熟悉了python的调试环境后发现python调试和C差不多吗。不错不错，^_^

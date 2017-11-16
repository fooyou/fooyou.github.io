---
layout: post
title: 在vim中使用vim[grep]检索工程目下的任意文本
category: Document
tags: vim
date: 2015-12-01 13:12:37
published: true
summary: sublime text中在某个工程目录右键有个在当前目录下搜索的功能，很实用，尤其在阅读代码的时候，那么在vim里怎么实现呢？
image: pirates.svg
comment: true
---

文本编辑器一个很使用的功能就是在对个文件中搜索文本（可以使用正则搜索），vim当然也有这个功能，只是不容易被发现它有这个功能，其实vim之所以能成为最好的文本编辑器，除了快捷外还有功能齐全和易扩展。

效果图如下：

![vim search demo](https://github.com/fooyou/fooyou.github.io/blob/master/img/posts/2015-12-01_11-15-29.gif?raw=true)

搜了一下发现vim wiki里有关于多个文件搜索内容的文章，比如[find in file within vim](http://vim.wikia.com/wiki/Find_in_files_within_Vim)，里面介绍了几个命令：

- :grep
- :lgrep
- :vimgrep
- :lvimgrep

在vim里运行`grep`和`lgrep`命令实际上是调用外部的命令在进行搜索，能处理复杂的正则表达式。

`vimgrep`和`lvimgrep`是vim内置的搜索命令，可以处理不太复杂的正则表达式。

它们的搜索结果都会放入一个列表里面。`grep`和`vimgrep`的搜索结果放在`quickfix list`里，quickfix list 可以使用`:cw`或者`:copen`在vim中打开，它的结果可以和所有的vim窗口共享；`lgrep`和`lvimgrep`的结果存放在`location list`里，location list 是当前窗口的，可以使用`:lw`或者`:lopen`打开。

最妙的是搜索结果在cw或lw展现的时候，可以回车跳转到指定文件的搜索文本的位置，这正是我想要的。

grep和lgrep依赖于外置命令，vimgrep和lvimgrep则随时可用，vimgrep声明如下：

```
: vim[grep][!] /{pattern}/[g][j] {file} ...
```

`g`：代表所有匹配，而不是其中的一行匹配
`j`：代表vim不会自动跳转到第一个匹配的地方

比如要在当前文件夹下递归所有文件搜索`tar`或者是`zip`，就可以这样搜：

```
: lvim /\<\(tar\|zip\)\>/gj **/*
: lw
```

## 应用场景

### 一、映射快捷键

tip: 使用`cword`取当前文件光标所在出的文字，`.vimrc`配置如下：

```
map <F3> :execute "lvimgrep /" . expand("<cword>") . "/gj **/*" <Bar> lw<CR>
```

上述配置完成后，在vim中当前光标下，按下F3就会在vim的当前目录下搜索所有的文件及其子文件夹的文件，并显示出来，还可以使用 %:e 来做，意思是当前目录（%）下的同类型文件（e），如下：

```
map <F3> :execute "lvimgrep /" . expand("<cword>") . "/gj " . expand("%:e") <Bar> lw<CR>
```

### 二、关闭autocmds以加速搜索

使用vimgrep搜索上百个文件会很慢，而用外置的grep就很快，一个原因是vimgrep使用vim的时序来读取文件，而这个时序将执行几个autocommands，所以我们在检索时关掉这个功能就会提速不少，所以最终的vimrc中配置如下：

```
map <F3> :noautocmd execute "lvimgrep /" . expand("<cword>") . "/gj **/*" <Bar> lw<CR>
```

当然，也可以直接执行 grep命令来提速。

> http://vim.wikia.com/wiki/Find_in_files_within_Vim 

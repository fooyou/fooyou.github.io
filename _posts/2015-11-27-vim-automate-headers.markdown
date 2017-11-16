---
layout: post
title: vim使用autocommand添加自定义文件头
category: Document
tags: vim
date: 2015-11-27 13:05:10
published: true
summary: sublime里有个插件叫fileheader非常赞，可为文件自动添加文件头和一些信息，在vim里也能简单实现。
image: pirates.svg
comment: true
latex: false
---


**_提要_**

vim 的 autocmd 命令可以在 vim 读写文件或进入离开缓冲区或者窗口的时候执行，从而可以使用`autocmd`命令对文件自动添加文件头，autocmd 声明如下：

```
autocmd {event} {pattern} {cmd}
```

**Events**：autocmd的events有40多个，为代码添加文件头，要用到BufNewFile，相关步骤如下：

## Step 1:

找个目录创建一个包含文件头内容的文本文件，注意开头第一行得是`:insert`，最后一行要有`.`，拿C文件的文件头内容为例：

```c
:insert
/**
 * @File Name:
 * @Author:
 * @Email:
 * @Create Date:
 * @Last Modified:
 * @Description:
 */
```

## Step 2: 在~/.vimrc中添加autocmd命令

在~/.vimrc中添加以下几行代码：

```
autocmd bufnewfile *.{cpp,c,h,cc,hpp,cs,js,go} so /home/joshua/Documents/fileheader/vim/C.tmpl
autocmd bufnewfile *.{cpp,c,h,cc,hpp,cs,js,go} exe "1," . 7 . "g/File Name:.*/s//File Name: " .expand("%")
autocmd bufnewfile *.{cpp,c,h,cc,hpp,cs,js,go} exe "1," . 7 . "g/Create Date:.*/s//Create Date: " .strftime("%Y-%m-%d %H:%m:%S")
autocmd Bufwritepre,filewritepre *.{cpp,c,h,cc,hpp,cs,js,go} execute "normal ma"
autocmd Bufwritepre,filewritepre *.{cpp,c,h,cc,hpp,cs,js,go} exe "1," . 7 . "g/Last Modified:.*/s//Last Modified: " .strftime("%Y-%m-%d %H:%m:%S")
autocmd bufwritepost,filewritepost *.{cpp,c,h,cc,hpp,cs,js} execute "normal `a"
```

## Step 3: 新建*.c文件将看到自动文件头

```
$ vim hello.c
```

将看到：

```c
/**
 * @File Name: hello.c
 * @Author: 
 * @Email: 
 * @Create Date: 2015-11-27 17:11:54
 * @Last Modified:
 * @Description:
 */
```

w保存后，将看到修改日期

```
/**
 * @File Name: hello.c
 * @Author:
 * @Email: 
 * @Create Date: 2015-11-27 17:11:54
 * @Last Modified: 2015-11-27 17:11:32
 * @Description:
 */
```

很棒啊！！，再把常用的几个添加一下。

参考：

> http://www.thegeekstuff.com/2008/12/vi-and-vim-autocommand-3-steps-to-add-custom-header-to-your-file/


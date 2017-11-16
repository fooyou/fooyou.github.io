---
layout: post
title: fbterm 下 w3m 不能显示图片的解决办法
category: Document
tags: linux w3m fbterm
date: 2016-06-27 16:06:16
published: true
summary: tty 使用 framebuffer 显示图片播放视频等，w3m 也可用其显示图片，但由于版本问题一直没有官方的更新。
image: pirates.svg
comment: true
---

## 问题的由来

使用 w3m 已经有两年的时间，由于经常需要切换到 tty 工作，所以就想把平时在桌面上能干的事都放在 tty 上。

由于版本问题 w3m 使用 framebuffer 显示图片时，指定的 fb 为 jfbterm（比较老的版本），可修改指定 fb 的代码 ./w3mimg/fb/fb_w3mimg.c 175 行的代码：

```
    --- if (!check_tty_console(getenv("W3M_TTY")) && strcmp("jfbterm", getenv("TERM")) != 0) {
    +++ if (!check_tty_console(getenv("W3M_TTY")) && (check_TERM(getenv("TERM")) == 0)) {
```

这样重新编译安装就 OK 了！


## 编译问题修改


w3m-0.5.3 的代码中的几个编译错误需要修改的：

1. istream.h/istream.c 的 struct 重复定义的错误，可以使用两种办法解决：

    a. 使用 sed 命令

    ```
    sed -i -e "s/file_handle/w3m_file_handle/g" istream.?
    ```

    b. 修改这两个文件中的结构体名 file_handle 为任何其他不重复的名字比如：io_file_handle，然后保存编译

2. make 时如果遇到 GC_proc_set_param() 返回值为 void 的错误，请修改文件 main.c，API 升级导致的接口错误：

    ```
    314: else if (orig_GC_warn_proc)
    修改为：
    314: else if (orig_GC_warn_proc = GC_get_warn_proc())

    863: orig_GC_warn_proc = GC_set_warn_proc(wrap_GC_warn_proc)
    修改为：
    863: GC_set_warn_proc(wrap_GC_warn_proc)
    ```

3. make 时如果遇到 ld 返回错误，请修改 Makefile.in 文件

    ```
    Index: w3m-0.5.3/Makefile.in
    ===================================================================
    --- w3m-0.5.3.orig/Makefile.in  2010-12-03 09:38:54.699796002 +0100
    +++ w3m-0.5.3/Makefile.in   2010-12-03 09:40:39.739796001 +0100
    @@ -193,7 +193,7 @@
        $(CC) $(CFLAGS) -DDUMMY -c -o $@ $?

     $(IMGDISPLAY): w3mimgdisplay.o $(ALIB) w3mimg/w3mimg.a
 -   $(CC) $(CFLAGS) -o $(IMGDISPLAY) w3mimgdisplay.o w3mimg/w3mimg.a $(LDFLAGS) $(LIBS) $(IMGLDFLAGS)
 +   $(CC) $(CFLAGS) -o $(IMGDISPLAY) w3mimgdisplay.o w3mimg/w3mimg.a $(LDFLAGS) $(LIBS) -lX11 $(IMGLDFLAGS)

      w3mimgdisplay.o: w3mimgdisplay.c w3mimg/w3mimg.h
      $(CC) $(CFLAGS) $(IMGCFLAGS) -o $@ -c $(srcdir)/w3mimgdisplay.c
    ```

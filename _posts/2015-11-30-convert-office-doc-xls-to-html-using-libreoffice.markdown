---
layout: post
title: 使用libreoffice把*.[odt|xls|doc|docx|xlsx]转化成html
category: Document
tags: doc office html
date: 2015-11-30 16:11:19
published: true
summary: 有许多还不知道的 linux 系统已经集成好的功能，比如 libreoffice 提供的这个功能，可以把 office 文档或者表格转化成 html。
image: pirates.svg
comment: true
---

> 需求驱动技术更新。

办公室文档全是Office（doc，ppt，excel），统治着Office市场的是 MS，当然开源的 OpenOffice 也不是吃素的虽然体验没有 MS 的好，但咱兼容啊，重要的是免费啊，但随着开源组织的壮大，OpenOffice 的体验也不断的在提升啊。（不过论体验 Mac OS 的也不错也兼容也免费）。

但码农门通常不喜欢 Office，尤其是开源社区的码农门，如果你让他们用 Office 写篇文档，他们思考的往往是这东西能有版本控制吗？我怎么能看到每一次提交的变化呢？所以对于他们来说 Office 不太适合他们作为写文档的工具，他们喜欢 Markdown，更有些有洁癖的 Peeker，也喜欢用 Graphviz 来画图，为什么？因为基于字符的文档不像二进制文件它们可以版本控制。所以对于一些复杂的文档，这些码农门也不愿意用Office，而是用各种可字符化的图形工具生成 HTML 文档以供查看。这样码农门可以像管理自己代码一样维护自己的文档，哪儿该更新，哪儿该修改一目了然。

但老板们喜欢 Office 啊，插个图，打个印，排个版，方便！但这两天拿到老板们的Office文档傻眼了，我的开发环境是tty啊，看个毛的 Office 啊。

google了一下，居然发现 libreoffice 封装了个 `soffice` 命令，可以把doc，odt，xls等转化成 html，这不就好了么。

```
soffice --headless --convert-to output_file_extension[:output_filter_name] [--outdir output_dir] files
```

headless: 去处文件头

output_file_extension[:output_filter_name]: 要生成的文件扩展和过滤器名，比如`htm:HTML`

比如要转换`xxxxxxx.docx`文档为html，就可以这样：

```
$ soffice --headless --convert-to html:HTML xxxxxxx.docx
```

这样就会在当前目录下生成一个名为`xxxxxxx.html`的文档，然后用`w3m`打开就可以阅读了，这样有点麻烦，但写个脚本就简单了。

这个命令有许多其他功能，有时间在挖掘吧。

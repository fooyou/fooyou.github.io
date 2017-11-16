---
layout: post
title: Markdwon中插入数学公式
category: Document
tags: markdown 
year: 2015
month: 08
day: 19
published: true
summary: 就是不愿意在markdown中引入图片，尤其数学公式，于是网搜了一下方法。
image: pirates.svg
comment: true
latex: true
---

去他的Office，自使用Markdown书写文档以来，一直乐在其中，只有一点就是导入图片的问题，像流程图啊，公式啊什么都得用其他图形工具做好后，然后上传到某个服务器上，然后在Markdown里引用。非常麻烦。不过话说回来，如果使用Office不也是这个过程吗？更何况Markdown可直接发布在网页上呢！

图形的问题可以用graphviz解决，程序员最不怕的就是敲代码，它可以只敲一些代码就可以把各种流程图什么的图形画出来，不怕乱的话，可以直接把生成的svg代码嵌入到markdown里，当然也可以路径引入。

至于公式吗，以下有三个解决方案：

## 方案一：使用Google Chart的服务器

Google Chart API:

```html
<img src="http://chart.googleapis.com/chart?cht=tx&chl= 在此插入Latex公式" style="border:none;">
```

一个例子：

```html
<img src="http://chart.googleapis.com/chart?cht=tx&chl=\Large x=\frac{-b\pm\sqrt{b^2-4ac}}{2a}" style="border:none;">
```

效果如下：

<img src="http://chart.googleapis.com/chart?cht=tx&chl=\Large x=\frac{-b\pm\sqrt{b^2-4ac}}{2a}" style="border:none;">

可是google api是墙外的，不保靠。并且其还是以<img>标签出现的，不利于主题的修改。

## 方案二：使用forkosh服务器

forkosh API:

```html
<img src="http://www.forkosh.com/mathtex.cgi? 在此处插入Latex公式">
```

例子：

```html
<img src="http://www.forkosh.com/mathtex.cgi? \Large x=\frac{-b\pm\sqrt{b^2-4ac}}{2a}">
```

显示结果：

<img src="http://www.forkosh.com/mathtex.cgi? \Large x=\frac{-b\pm\sqrt{b^2-4ac}}{2a}">


## 方案三：使用MathJax引擎

用这个引擎生成的公式不是img，这就是我想要的，可以这样添加公式引擎：

```html
<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default"></script>
```

然后写入Tex公式，`$$公式$$`：表示行间公式；`\\(公式\\)`：行内公式，一个简单例子：

```
$$x=\frac{-b\pm\sqrt{b^2-4ac}}{2a}$$
\\(x=\frac{-b\pm\sqrt{b^2-4ac}}{2a}\\)
```

显示效果如下：

行间：

$$x=\frac{-b\pm\sqrt{b^2-4ac}}{2a}$$

行内：

\\(x=\frac{-b\pm\sqrt{b^2-4ac}}{2a}\\)


这就是你想要的有木有？很帅，有木有？！

> 参考： http://blog.csdn.net/xiahouzuoxin/article/details/26478179

有关LaTeX公式的写法，请参照：

> https://zh.wikibooks.org/wiki/LaTeX/%E6%95%B0%E5%AD%A6%E5%85%AC%E5%BC%8F



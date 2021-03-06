---
layout: post
title: jekyll博客的分页实现
category: Document
tags: jekyll blog
date: 2015-07-24
published: true
summary: jekyll本身支持分页功能
mathjax: false
highlight: true
---

> 引用：http://segmentfault.com/a/1190000000406015

## 开启分页功能
------------

首先我们需要在jekyll中开启分页功能，在jekyll的_config.yml中加入分页配置：

```vim
paginate: 5
paginate_path: "page:num"
```

第一行定义了每页的文章数量，而第二行则定义了在分页的结果，比如在/index.html中使用分页，定义为page:num，则第二页的路径将是/page2/index.html，第三页的路径将是/page3/index.html，以此类推

_需要注意的几个点：_

* 分页只在html文件中起作用
* paginate_path同时定义了需要被分页的文件，本人测试这个叫index.html，具体目录由paginate_path中的路径定义，如果定义的目录没有，则会向上寻找index.html，直到根目录的index.html，具体机制官网上没有详细说，所以还需要进一步实验

## 使用分页
------------

只是开启了分页还没有用，我们需要确实使用到首页之中，在首页(/index.html)中添加如下代码：

```html
{% for post in paginator.posts %}
    <a href="{{ post.url }}">{{ post.title }}</a>
{% endfor %}
```

这样，jekyll就会根据paginator来进行分页了，被分出来多少页，就会有多少个页面生成。排1-5的文章就在/index.html中了，而排6-10的文章则在/page2/index.html中，依次类推

## 换页
-------------

只是分页还不够，我们还需要在每个页面上做一个跳转到其他页面的导航，这里需要用到paginator的一些其他属性

首先检测总的页数，如果只有一页，自然就不需要分页了。通过paginator的total_pages属性能判断总页数：

```html
{% if paginator.total_pages > 1 %}
<!-- 分页代码 -->
{% endif %}
```

我们需要一个跳转到上一页的按钮，这个按钮在第一页不需要显示，通过paginator的previous\_page属性来判断是否是第一个页面，使用paginator的previous\_page\_path来输出上一页的路径，注意在前面添加baseurl，并进行一些必要的字符替换：

```html
{% if paginator.previous_page %}
    <a href="{{ paginator.previous_page_path | prepend: site.baseurl | replace: '//', '/' }}"上一页</a>
{% endif %}
```

接着是生成所有页面的按钮，并使当前页按钮无效化，遍历所有页面，使用paginator的page属性来确定当前页，如果是当前页，则按钮无效，否则使用{{ site.paginate_path | prepend: site.baseurl | replace: '//', '/' | replace: ':num', page }}来将:num替换成当前页面的数字生成页面路径：

```html
{% for page in (1..paginator.total_pages) %}
    {% if page == paginator.page %}
      <span class="active">{{ page }}</span>
    {% elsif page == 1 %}
      <a href="{{ '/index.html' | prepend: site.baseurl | replace: '//', '/' }}">{{ page }}</a>
    {% else %}
      <a href="{{ site.paginate_path | prepend: site.baseurl | replace: '//', '/' | replace: ':num', page }}">{{ page }}</a>
    {% endif %}
{% endfor %}
```

最后生成一个下一页的按钮，在最后一页不显示，和上一页按钮类似，通过paginator的next_page_path来确定是否还有下一页：

```html
{% if paginator.next_page %}
    <a href="{{ paginator.next_page_path | prepend: site.baseurl | replace: '//', '/' }}">下一页</a>
{% endif %}
```

这样一个完整的分页导航功能就做好了，效果就和博客主页上的分页效果是一样的。

## 总结

jekyll的分页总的来说还算给力，基本的功能可以完成。但是有一些缺陷，就是category，tag的分类分页无法实现，必须通过插件的方式来做。但是github不允许三方插件，所以只能通过一些其他的方式实现了。

> 引用：http://segmentfault.com/a/1190000000406015

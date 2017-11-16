---
layout: post
title: Scrapy 学习笔记（一）
category: Document
tags: python scrapy
year: 2015
month: 04
day: 24
published: true
summary: 妹呀，早知道有scrapy就不用BeautifulSoup自己解析response了，快来学习吧，简单，高效
image: pirates.svg
comment: true
---

本文以dmoz.org网站为例，介绍简单的scrapy爬虫的创建。

<br>
<br>
## 创建项目
Scrapy必须要创建项目先，然后在项目目录下方能工作。如下命令创建：

```
$ scrapy startproject tutorial
```

之后，你能看到新建的scrapy工程目录：

```
└── tutorial
    ├── scrapy.cfg			：项目的配置文件
    └── tutorial			：该项目的Python模块。之后将在此加入代码。
        ├── __init__.py
        ├── items.py		：项目中的item文件。
        ├── pipelines.py	：项目中的piplines文件。
        ├── settings.py		：项目的设置文件。
        └── spiders			：放置spider代码的目录
            └── __init__.py

```

<br>
<br>
## 定义Item

*Item* 是保存爬取到的数据的容器；其使用方法和python字典类似， 并且提供了额外保护机制来避免拼写错误导致的未定义字段错误。

首先根据需要从dmoz.org获取到的数据对item进行建模。 我们需要从dmoz中获取名字，url，以及网站的描述。 对此，在item中定义相应的字段。编辑 tutorial 目录中的 items.py 文件:

```python
import scrapy

class DmozItem(scrapy.Item):
    title = scrapy.Field()
    link = scrapy.Field()
    desc = scrapy.Field()
```

<br>
<br>
## 编写第一个爬虫（Spider）

Spider是用户编写用于从单个网站(或者一些网站)爬取数据的类。

其包含了一个用于下载的初始URL，如何跟进网页中的链接以及如何分析页面中的内容， 提取生成 `item` 的方法。

为了创建一个Spider，您必须继承 `scrapy.Spider` 类， 且定义以下三个属性:

- `name`：用于区别Spider。 该名字必须是唯一的，您不可以为不同的Spider设定相同的名字。
- `start_urls`：包含了Spider在启动时进行爬取的url列表。 因此，第一个被获取到的页面将是其中之一。 后续的URL则从初始的URL获取到的数据中提取。
- `parse()`：是spider的一个方法。 被调用时，每个初始URL完成下载后生成的 `Response`对象将会作为唯一的参数传递给该函数。 该方法负责解析返回的数据(response data)，提取数据(生成item)以及生成需要进一步处理的URL的 `Request` 对象。

以下为我们的第一个Spider代码，保存在 `tutorial/spiders` 目录下的 `dmoz_spider.py` 文件中:

```python
import scrapy

class DmozSpider(scrapy.Spider):
    name = "dmoz"
    allowed_domains = ["dmoz.org"]
    start_urls = [
        "http://www.dmoz.org/Computers/Programming/Languages/Python/Books/",
        "http://www.dmoz.org/Computers/Programming/Languages/Python/Resources/"
    ]

    def parse(self, response):
        filename = response.url.split("/")[-2]
        with open(filename, 'wb') as f:
            f.write(response.body)
```

<br>
<br>
## 爬取

进入项目的根目录，执行下列命令启动spider:

```
scrapy crawl dmoz
```

`crawl dmoz` 启动用于爬取 `dmoz.org` 的spider，您将得到类似的输出:

```
2015-04-24 15:45:12+0800 [scrapy] INFO: Scrapy 0.24.6 started (bot: tutorial)
2015-04-24 15:45:12+0800 [scrapy] INFO: Optional features available: ssl, http11, django
2015-04-24 15:45:12+0800 [scrapy] INFO: Overridden settings: {'NEWSPIDER_MODULE': 'tutorial.spiders', 'SPIDER_MODULES': ['tutorial.spiders'], 'BOT_NAME': 'tutorial'}
2015-04-24 15:45:12+0800 [scrapy] INFO: Enabled extensions: LogStats, TelnetConsole, CloseSpider, WebService, CoreStats, SpiderState
2015-04-24 15:45:12+0800 [scrapy] INFO: Enabled downloader middlewares: HttpAuthMiddleware, DownloadTimeoutMiddleware, UserAgentMiddleware, RetryMiddleware, DefaultHeadersMiddleware, MetaRefreshMiddleware, HttpCompressionMiddleware, RedirectMiddleware, CookiesMiddleware, HttpProxyMiddleware, ChunkedTransferMiddleware, DownloaderStats
2015-04-24 15:45:12+0800 [scrapy] INFO: Enabled spider middlewares: HttpErrorMiddleware, OffsiteMiddleware, RefererMiddleware, UrlLengthMiddleware, DepthMiddleware
2015-04-24 15:45:12+0800 [scrapy] INFO: Enabled item pipelines: 
2015-04-24 15:45:12+0800 [dmoz] INFO: Spider opened
2015-04-24 15:45:12+0800 [dmoz] INFO: Crawled 0 pages (at 0 pages/min), scraped 0 items (at 0 items/min)
2015-04-24 15:45:12+0800 [scrapy] DEBUG: Telnet console listening on 127.0.0.1:6023
2015-04-24 15:45:12+0800 [scrapy] DEBUG: Web service listening on 127.0.0.1:6080
2015-04-24 15:45:13+0800 [dmoz] DEBUG: Crawled (200) <GET http://www.dmoz.org/Computers/Programming/Languages/Python/Resources/> (referer: None)
2015-04-24 15:45:14+0800 [dmoz] DEBUG: Crawled (200) <GET http://www.dmoz.org/Computers/Programming/Languages/Python/Books/> (referer: None)
2015-04-24 15:45:14+0800 [dmoz] INFO: Closing spider (finished)
2015-04-24 15:45:14+0800 [dmoz] INFO: Dumping Scrapy stats:
    {'downloader/request_bytes': 516,
     'downloader/request_count': 2,
     'downloader/request_method_count/GET': 2,
     'downloader/response_bytes': 16670,
     'downloader/response_count': 2,
     'downloader/response_status_count/200': 2,
     'finish_reason': 'finished',
     'finish_time': datetime.datetime(2015, 4, 24, 7, 45, 14, 17440),
     'log_count/DEBUG': 4,
     'log_count/INFO': 7,
     'response_received_count': 2,
     'scheduler/dequeued': 2,
     'scheduler/dequeued/memory': 2,
     'scheduler/enqueued': 2,
     'scheduler/enqueued/memory': 2,
     'start_time': datetime.datetime(2015, 4, 24, 7, 45, 12, 133706)}
2015-04-24 15:45:14+0800 [dmoz] INFO: Spider closed (finished)
```

查看包含 [dmoz] 的输出，可以看到输出的log中包含定义在 start_urls 的初始URL，并且与spider中是一一对应的。在log中可以看到其没有指向其他页面( (referer:None) )。

除此之外，更有趣的事情发生了。就像我们 parse 方法指定的那样，有两个包含url所对应的内容的文件被创建了: Book , Resources 。

** 刚才发生了什么 **

Scrapy为Spider的 `start_urls` 属性中的每个URL创建了 `scrapy.Request` 对象，并将 `parse` 方法作为回调函数(callback)赋值给了Request。

Request对象经过调度，执行生成 `scrapy.http.Response` 对象并送回给spider parse() 方法。

<br>
<br>
## 提取Item

**Selectors选择器简介**

从网页中提取数据有很多方法。Scrapy使用了一种基于 `XPath` 和 `CSS` 表达式机制: `Scrapy Selectors` 。 关于selector和其他提取机制的信息请参考 Selector文档 。

这里给出XPath表达式的例子及对应的含义:

- `/html/head/title:` 选择HTML文档中 `<head>` 标签内的 `<title>` 元素
- `/html/head/title/text():` 选择上面提到的 `<title>` 元素的文字
- `//td:` 选择所有的 `<td>` 元素
- `//div[@class="mine"]:` 选择所有具有 `class="mine"` 属性的 `div` 元素

上边仅仅是几个简单的XPath例子，XPath实际上要比这远远强大的多。 如果您想了解的更多，我们推荐 这篇[XPath教程](http://www.w3schools.com/XPath/default.asp) 。

为了配合XPath，Scrapy除了提供了 Selector 之外，还提供了方法来避免每次从response中提取数据时生成selector的麻烦。

Selector有四个基本的方法(点击相应的方法可以看到详细的API文档):

- `xpath()`: 传入xpath表达式，返回该表达式所对应的所有节点的selector list列表 。
- `css()`: 传入CSS表达式，返回该表达式所对应的所有节点的selector list列表.
- `extract()`: 序列化该节点为unicode字符串并返回list。
- `re()`: 根据传入的正则表达式对数据进行提取，返回unicode字符串list列表。

在查看了网页的源码后，您会发现网站的信息是被包含在 第二个 <ul> 元素中。

我们可以通过这段代码选择该页面中网站列表里所有 <li> 元素:

```
sel.xpath('//ul/li')
```

网站的描述:

```
sel.xpath('//ul/li/text()').extract()
```

网站的标题:

```
sel.xpath('//ul/li/a/text()').extract()
```

以及网站的链接:

```
sel.xpath('//ul/li/a/@href').extract()
```

之前提到过，每个 .xpath() 调用返回selector组成的list，因此我们可以拼接更多的 .xpath() 来进一步获取某个节点。我们将在下边使用这样的特性:

```python
for sel in response.xpath('//ul/li'):
    title = sel.xpath('a/text()').extract()
    link = sel.xpath('a/@href').extract()
    desc = sel.xpath('text()').extract()
    print title, link, desc
```

在我们的spider中加入这段代码:

```
import scrapy

class DmozSpider(scrapy.Spider):
    name = "dmoz"
    allowed_domains = ["dmoz.org"]
    start_urls = [
        "http://www.dmoz.org/Computers/Programming/Languages/Python/Books/",
        "http://www.dmoz.org/Computers/Programming/Languages/Python/Resources/"
    ]

    def parse(self, response):
        for sel in response.xpath('//ul/li'):
            title = sel.xpath('a/text()').extract()
            link = sel.xpath('a/@href').extract()
            desc = sel.xpath('text()').extract()
            print title, link, desc
```

现在尝试再次爬取dmoz.org，您将看到爬取到的网站信息被成功输出:

```
scrapy crawl dmoz
```

## 使用Item


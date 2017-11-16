---
layout: post
title: Mac os 配置代理的 pac 文件介绍
category: Document
tags: pac proxy
date: 2015-11-28 00:11:54
published: true
summary: pac文件是什么，怎么通过pac文件针对不同host设置不同代理？
image: pirates.svg
comment: true
---

## 什么是pac文件

PAC：Proxy Auto-Configuration的缩写，pac文件使用javascript编写可根据不同的url或者host选择不同的访问策略。PAC文件可以控制浏览器如何处理HTTP、HTTPS、和FTP请求。

它是由netscape很早之前提出的，现在主流系统和浏览器都支持PAC文件了，有它就不用再为各个用户代理单独维护了。

## 编写

其函数入口为 `FindProxyForURL(url, host)`，用户代理调用此函数，并根据返回值决定如何访问指定URL。

```js
function FindProxyForURL(url, host) {
    return 'DIRECT';
}
```

url: 将要访问的网络地址，比如www.google.com/news

host：url的host，比如www.google.com

PAC 文件中还定义了一些函数，用于url host ip解析之类的，比如：

- isPlainHostName(host)
- dnsDomainIs(host, domain)
- shExpMatch(str, pattern)

不过我的pac文件就用了dnsDomainIs这一个函数，如下：

proxy.pac

```js
function FindProxyForURL(url, host) {
    ow = [
        ".google.com", ".googlevideo.com",
        ".youtube.com", ".ytimg.com", ".ggpht.com", ".googlesyndication.com", ".youtube-nocookie.com"
        ".facebook.com",
        ".twitter.com", ".twimg.com"
    ];
    for (i = 0; i < ow.length; i++) {
        if (dnsDomainIs(host, ow[i])) {
            return "SOCKS 127.0.0.1:7777";
        }
    }
	return "DIRECT";
}
```

编写好后，存在某个地方，然后配置在 系统偏好设置-> 网络 -> Wifi -> 高级 -> 代理 -> 自动代理配置 -> URL 里就可以了。

其实不用开local server或者把pac文件放在apache下，直接使用file://协议就可以比如我的配置路径在 `/User/joshua/Document/proxy.pac`

配置URL路径为：

```
URL：file:///User/joshua/Document/proxy.pac
```

就好用了！

## 提示

- PAC文件对于私有IP网络，内部host和.local域名的host是忽略的

参见：

> http://findproxyforurl.com/example-pac-file/


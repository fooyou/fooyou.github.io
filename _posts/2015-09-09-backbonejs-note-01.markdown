---
layout: post
title: Backbone简介
category: Document
tags: backbone
date: 2015-09-09 11:11:13
published: true
summary: Backbone概要介绍
image: pirates.svg
comment: true
latex: false
---

  术语  |  描述
--------|----------
 DOM    | 文档对象模型
 MVC    | 模型-视图-控制器
 SPA    | 单页面应用

## 前言

Web 应用程序越来越关注于前端，使用客户端脚本与 Ajax 进行交互。由于 JavaScript 应用程序越来越复杂，如果没有合适的工具和模式，那么 JavaScript 代码的高效编写、非重复性和可维护性方面会面临挑战。模型-视图-控制器 (MVC) 是一个常见模式，可用于服务器端开发以生成有组织以及易维护的代码。MVC 支持将数据（比如通常用于 Ajax 交互的 JavaScript Object Notation (JSON) 对象）从表示层或从页面的文档对象模型 (document object model, DOM) 中分离出来，也可适用于客户端开发。

Backbone.js是轻量级的js框架库，它提供了一套web开发的框架，可用于创建MVC类应用程序。Backbone：

- 通过Models进行key-value绑定及自定义事件处理；
- 通过Collections提供一套丰富的API用于枚举功能；
- 通过Views来进行事件处理及与现有的Application通过RESTful JSON接口进行交互；
- 强制依赖underscore.js，非强制依赖jQuery。

Backbone能让你像写Java（后端）代码组织js代码，定义类，类的属性以及方法。更重要的是它能够优雅的把原本无逻辑的javascript代码进行组织，并且提供数据和逻辑相互分离的方法，减少代码开发过程中的数据和逻辑混乱。

在Backbonejs有几个重要的概念，先介绍一下:Model，Collection，View，Router。其中Model是根据现实数据建立的抽象，比如人（People）；Collection是Model的一个集合，比如一群人；View是对Model和Collection中数据的展示，把数据渲染（Render）到页面上；Router是对路由的处理，就像传统网站通过url现实不同的页面，在单页面应用（SPA）中通过Router来控制前面说的View的展示。

通过Backbone，你可以把你的数据当作Models，通过Models你可以创建数据，进行数据验证，销毁或者保存到服务器上。当界面上的操作引起model中属性的变化时，model会触发change的事件。那些用来显示model状态的views会接受到model触发change的消息，进而发出对应的响应，并且重新渲染新的数据到界面。在一个完整的Backbone应用中，你不需要写那些胶水代码来从DOM中通过特殊的id来获取节点，或者手工的更新HTML页面，因为在model发生变化时，views会很简单的进行自我更新。

**Model和View**

![Model_View](http://backbonejs.org/docs/images/intro-model-view.svg)

**Collections**

![Collections](http://backbonejs.org/docs/images/intro-collections.svg)

**View渲染**

![View_Rendering](http://backbonejs.org/docs/images/intro-views.svg)

**路由**

![Routing_URLs](http://backbonejs.org/docs/images/intro-routing.svg)

*个人感受*

感觉Backbone为前端开发定制了一套规则，在此规则下开发者可以像使用django组织python代码一样组织js代码，Backbone很优雅，使得前端和Server交互更加简单。

关于backbone的更多介绍和学习资料：

> http://backbonejs.org/

> http://backbonetutorials.com/


## backbone的应用范围

其适用场景非常广，无论Web-Page还是Web-App都可以应用，最佳应用场合还是SPA，但使用前要充分理解Backbone的设计思想。

Backbone的优点和一些经验Tip：

- 代码简洁，分层清晰，质量比较高，只做框架该做的事，很容易和其他工具或框架整合，对于前端开发者通读一遍好处大大的。
- MVC架构清晰，易于了解整体设计
- View的划分将页面上的视图元素解耦，粒度细化。View间通过事件和Model通讯，避免了DOM事件的滥用。
- Model和Restful的通讯方式对于后端人员非常友好。
- Collections/Model抽象了以前杂乱的AJAX请求，CRUD请求变得非常方便。
- Tip：强烈建议 View 到 Model 单向依赖。
- Tip：可以和其他一些框架或工具联合使用，如：requirejs、SeaJS、Potjs等

Backbone的一些缺点或者说尚未实现的Feature：

- Model层稍简单（和django等相比），像一些复杂的one-to-one或者one-to-many等复杂数据关系力不从心。另外一个Model只能属于一个Collection设计，页面复杂的时候很受局限。（PS：[Backbone.Relations](https://github.com/PaulUithol/Backbone-relational)插件是这个问题的一个解决方案）
- Model只有基本的CRUD操作，不能很好的扩展，Backbone.sync方法写的不太灵活，要扩展就得重写sync方法。
- View层没有很强的Page管理机制，比如通过URL切换改变整个页面时，页面上尚存的View如何处理？直接销毁的话，是否要销毁关联的Model、Collection？Cache住？那么又如何Cache，是很烦人的问题。
- 内存管理需要谨慎，Backbone缺乏机制来避免创建重复Model，必要时需要自己编写Model管理。
- 扩展重写父类方法时，得写一大堆SuperClass.prototype.someMethod.apply之类的，因为其没有实现_super方法。
- Backbone的作者有代码洁癖（是好事也可能是坏事），像this.$el是大家呼唤很久才给加上的

总之Backbone是非常优雅的轻量级js框架，如果有些地方不习惯，你可以来扩展。

国内使用Backbone的一些网站：

1. [豆瓣阿尔法城](http://alphatown.com/)
2. [豆瓣阅读](http://read.douban.com/)
3. [百度开发者中心](http://developer.baidu.com/)

## 一个完整Demo

这个demo的主要功能是点击页面上得“新手报到”按钮，弹出对话框，输入内容之后，把内容拼上固定的字符串显示到页面上。事件触发的逻辑是： click 触发checkIn方法，然后checkIn构造World对象放到已经初始化worlds这个collection中。

完整代码

```html
<!DOCTYPE html>
<html>
<head>
    <title>backbone.js-Hello World</title>
    <meta charset="utf-8">
</head>
<body>
    <button id="check">新手报到</button>
    <ul id="world-list">
    </ul>
    <script src="http://backbonejs.org/test/vendor/jquery.js"></script>
    <script src="http://backbonejs.org/test/vendor/underscore.js"></script>
    <script src="http://backbonejs.org/backbone.js"></script>
    <script>
    (function ($) {
        World = Backbone.Model.extend({
            //创建一个World的对象，拥有name属性
            name: null
        });

        Worlds = Backbone.Collection.extend({
            //World对象的集合
            initialize: function (models, options) {
                    this.bind("add", options.view.addOneWorld);
            }
        });

        AppView = Backbone.View.extend({
            el: $("body"),
            initialize: function () {
                //构造函数，实例化一个World集合类
                //并且以字典方式传入AppView的对象
                this.worlds = new Worlds(null, { view : this })
            },
            events: {
                //事件绑定，绑定Dom中id为check的元素
                "click #check":  "checkIn",
            },
            checkIn: function () {
                var world_name = prompt("请问，您是哪星人?");
                if(world_name == "") world_name = '未知';
                var world = new World({ name: world_name });
                this.worlds.add(world);
            },
            addOneWorld: function(model) {
                $("#world-list").append("<li>这里是来自 <b>" + model.get('name') + "</b> 星球的问候：hello world！</li>");
            }
        });
        //实例化AppView
        var appview = new AppView;
    })(jQuery);
    </script>
</body>
</html>
```

这里面涉及到backbone的三个部分，View、Model、Collection，其中Model代表一个数据模型，Collection是模型的一个集合，而View是用来处理页面以及简单的页面逻辑的。

动手把代码放到你的编辑器中吧，成功执行，然后修改某个地方，再次尝试。


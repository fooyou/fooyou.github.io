---
layout: post
title: Backbone Sample todos分析
category: Document
tags: backbone
date: 2015-09-14 11:15:57
published: true
summary:
image: pirates.svg
comment: true
latex: false
---

经过前面的几篇文章，Backbone中的Model, Collection，Router，View，都简单的介绍了一下，我觉得看完这几篇文章，差不多就能开始使用Backbone来做东西了，所有的项目无外乎对这几个模块的使用。不过对于实际项目经验少些的同学，要拿起来用估计会有些麻烦。因此这里就先找个现成的案例分析一下。

## 大家都来分析todos

关于Backbonejs实例分析的文章网上真是一搜一大把，之所以这么多，第一是这东西需求简单，不用花时间到理解情景中；第二是代码就是官方的案例，顺手可得，也省得去找了，自己琢磨一个不得花时间吗。

于是就有人问了，丫们都在分析todos，能不能有点新意呢。这问题要我说，如果你真的能把todos搞明白了，那其他的也就不用去看了。不管是看谁的分析，把这个搞明白的。所有的项目大体思路都差不多。尤其是对于这样的MVC的模型，就是往对应的模块里填东西。因此，不管有多少人都在分析这玩意，自己弄懂了才是应该关心的。

话虽如此，不同于网络上的绝大部分的分析的是，the5fire除了分析这个，还是对其进行了扩充，另外在后面也会有真实的案例。但我也是从这些案例的代码中汲取的营养。

补充一下，新版的todos代码相较于之前简直清晰太多，完全可以当做一个前端的范本来学习、模仿。

## 获取代码

todos的代码这里下载：https://github.com/jashkenas/backbone/ ，建议自己clone一份到本地。线上的地址是：http://backbonejs.org/examples/todos/index.html

clone下来之后可以在example中找到todos文件夹，文件结构如下：:

```
examples
├── backbone.localStorage.js
└── todos
    ├── destroy.png
    ├── index.html
    ├── todos.css
    └── todos.js
```

1 directory, 5 files
用浏览器打开index.html文件，推荐使用chome浏览器，就可以看到和官网一样的界面了。关键代码都在todos.js这个文件里。

## 功能分析

首先来分析下页面上有哪些功能:

- 任务管理
    - 添加任务
    - 修改任务
    - 删除任务
- 统计
    - 任务总计
    - 已完成数目

总体上就这几个功能。

这个项目仅仅是在web端运行的，没有服务器进行支持，因此在项目中使用了一个叫做backbone-localstorage的js库，用来把数据存储到前端。

## 从模型下手

因为Backbone为MVC模式，根据对这种模式的使用经验，我们从模型开始分析。首先我们来看Model部分的代码:

```js
/**
*基本的Todo模型，属性为：title,order,done。
**/
var Todo = Backbone.Model.extend({
    // 设置默认的属性
    defaults: {
        title: "empty todo...",
        order: Todos.nextOrder(),
        done: false
    },

    // 设置任务完成状态
    toggle: function() {
        this.save({done: !this.get("done")});
    }
});
```

这段代码是很好理解的，不过我依然是画蛇添足的加上了一些注释。这个Todo显然就是对应页面上的每一个任务条目。那么显然应该有一个collection来统治（管理）所有的任务，所以再来看collection：

```js
/**
*Todo的一个集合，数据通过localStorage存储在本地。
**/

var TodoList = Backbone.Collection.extend({

    // 设置Collection的模型为Todo
    model: Todo,
    //存储到浏览器，以todos-backbone命名的空间中
    //此函数为Backbone插件提供
    //地址：https://github.com/jeromegn/Backbone.localStorage
    localStorage: new Backbone.LocalStorage("todos-backbone"),

    //获取所有已经完成的任务数组
    done: function() {
        return this.where({done: true});
    },

    //获取任务列表中未完成的任务数组
    //这里的where在之前是没有的，但是语法上更清晰了
    //参考文档：http://backbonejs.org/#Collection-where
    remaining: function() {
        return this.where({done: false});
    },

    //获得下一个任务的排序序号，通过数据库中的记录数加1实现。
    nextOrder: function() {
        if (!this.length) return 1;

        // last获取collection中最后一个元素
        return this.last().get('order') + 1;
    },

    //Backbone内置属性，指明collection的排序规则。
    comparator: 'order'
});
```

collection的主要功能有以下几个：

1. 获取完成的任务;
2. 获取未完成的任务;
3. 获取下一个要插入数据的序号;
4. 按序存放Todo对象。

## 为什么要两个view

首先要分析下，这俩view是用来干嘛的。有人可能会问了，这里不就是一个页面吗？一个view掌控全局不就完了？

我觉得这就是新手和老手的主要区别之一，喜欢在一个方法里面搞定一切，喜欢把东西都拧到一块去，觉得这样看起来容易。熟不知，这样的代码对于日后的扩展会造成很大的麻烦。因此我们需要学习下优秀的设计，从好的代码中汲取营养。

这里面的精华就是，将对数据的操作和对页面的操作进行分离，也就是现在代码里面TodoView和AppView。前者的作用是对把Model中的数据渲染到模板中;后者是对已经渲染好的数据进行处理。两者各有分工，TodoView可以看做是加工后的数据，这个数据就是待使用的html数据。

## TodoView的代码分析

TodoView是和Model一对一的关系，在页面上一个View也就展示为一个item。除此之外，每个view上还有其他的功能，比如编辑模式，展示模式，还有对用户的输入的监听。详细还是来看下代码：

```js
// 首先是创建一个全局的Todo的collection对象

var Todos = new TodoList;

// 先来看TodoView，作用是控制任务列表
var TodoView = Backbone.View.extend({

    //下面这个标签的作用是，把template模板中获取到的html代码放到这标签中。
    tagName:  "li",

    // 获取一个任务条目的模板,缓存到这个属性上。
    template: _.template($('#item-template').html()),

    // 为每一个任务条目绑定事件
    events: {
        "click .toggle"   : "toggleDone",
        "dblclick .view"  : "edit",
        "click a.destroy" : "clear",
        "keypress .edit"  : "updateOnEnter",
        "blur .edit"      : "close"
    },

    //在初始化时设置对model的change事件的监听
    //设置对model的destroy的监听，保证页面数据和model数据一致
    initialize: function() {
        this.listenTo(this.model, 'change', this.render);
        //这个remove是view的中的方法，用来清除页面中的dom
        this.listenTo(this.model, 'destroy', this.remove);
    },

    // 渲染todo中的数据到 item-template 中，然后返回对自己的引用this
    render: function() {
        this.$el.html(this.template(this.model.toJSON()));
        this.$el.toggleClass('done', this.model.get('done'));
        this.input = this.$('.edit');
        return this;
    },

    // 控制任务完成或者未完成
    toggleDone: function() {
        this.model.toggle();
    },

    // 修改任务条目的样式
    edit: function() {
        $(this.el).addClass("editing");
        this.input.focus();
    },

    // 关闭编辑模式，并把修改内容同步到Model和界面
    close: function() {
        var value = this.input.val();
        if (!value) {
            //无值内容直接从页面清除
            this.clear();
        } else {
            this.model.save({title: value});
            this.$el.removeClass("editing");
        }
    },

    // 按下回车之后，关闭编辑模式
    updateOnEnter: function(e) {
      if (e.keyCode == 13) this.close();
    },

    // 移除对应条目，以及对应的数据对象
    clear: function() {
        this.model.destroy();
    }
});
```

## AppView的代码分析

再来看AppView，功能是显示所有任务列表，显示整体的列表状态（如：完成多少，未完成多少）

```js
//以及任务的添加。主要是整体上的一个控制
var AppView = Backbone.View.extend({

    //绑定页面上主要的DOM节点
    el: $("#todoapp"),

    // 在底部显示的统计数据模板
    statsTemplate: _.template($('#stats-template').html()),

    // 绑定dom节点上的事件
    events: {
        "keypress #new-todo":  "createOnEnter",
        "click #clear-completed": "clearCompleted",
        "click #toggle-all": "toggleAllComplete"
    },

    //在初始化过程中，绑定事件到Todos上，
    //当任务列表改变时会触发对应的事件。
    //最后从localStorage中fetch数据到Todos中。
    initialize: function() {
        this.input = this.$("#new-todo");
        this.allCheckbox = this.$("#toggle-all")[0];

        this.listenTo(Todos, 'add', this.addOne);
        this.listenTo(Todos, 'reset', this.addAll);
        this.listenTo(Todos, 'all', this.render);

        this.footer = this.$('footer');
        this.main = $('#main');

        Todos.fetch();
    },

    // 更改当前任务列表的状态
    render: function() {
        var done = Todos.done().length;
        var remaining = Todos.remaining().length;

        if (Todos.length) {
            this.main.show();
            this.footer.show();
            this.footer.html(this.statsTemplate({done: done, remaining: remaining}));
        } else {
            this.main.hide();
            this.footer.hide();
        }

        //根据剩余多少未完成确定标记全部完成的checkbox的显示
        this.allCheckbox.checked = !remaining;
    },

    // 添加一个任务到页面id为todo-list的div/ul中
    addOne: function(todo) {
        var view = new TodoView({model: todo});
        this.$("#todo-list").append(view.render().el);
    },

    // 把Todos中的所有数据渲染到页面,页面加载的时候用到
    addAll: function() {
        Todos.each(this.addOne, this);
    },

    //生成一个新Todo的所有属性的字典
    newAttributes: function() {
        return {
            title this.input.val(),
            order:   Todos.nextOrder(),
            done:    false
        };
    },

    //创建一个任务的方法，使用backbone.collection的create方法。
    //将数据保存到localStorage,这是一个html5的js库。
    //需要浏览器支持html5才能用。
    createOnEnter: function(e) {
        if (e.keyCode != 13) return;
        if (!this.input.val()) return;

        //创建一个对象之后会在backbone中动态调用Todos的add方法
        //该方法已绑定addOne。
        Todos.create({title: this.input.val()});
        this.input.val('');
    },

    //去掉所有已经完成的任务
    clearCompleted: function() {
        // 调用underscore.js中的invoke方法
        //对过滤出来的todos调用destroy方法
        _.invoke(Todos.done(), 'destroy');
        return false;
    },

    //处理页面点击标记全部完成按钮
    //处理逻辑：
    //    如果标记全部按钮已选，则所有都完成
    //    如果未选，则所有的都未完成。
    toggleAllComplete: function () {
        var done = this.allCheckbox.checked;
        Todos.each(function (todo) { todo.save({'done': done}); });
    }
});
```

通过上面的代码，以及其中的注释，我们认识里面每个方法的作用。下面来看最重要的，页面部分。

## 页面模板分析

在前几篇的view介绍中我们已经认识过了简单的模板使用，以及变量参数的传递，如：

```html
<script type="text/template" id="search_template">

        <label><%= search_label %></label>
        <input type="text" id="search_input" />
        <input type="button" id="search_button" value="Search" />

</script>
```

既然能定义变量，那么就能使用语法，如同django模板，那来看下带有语法的模板，也是上面的两个view用到的模板，我想这个是很好理解的。

```html
<script type="text/template" id="item-template">
    <div class="view">
        <input class="toggle" type="checkbox" <%= done ? 'checked="checked"' : '' %> />
        <label><%- title %></label>
        <a class="destroy"></a>
    </div>
    <input class="edit" type="text" value="<%- title %>" />
</script>


<script type="text/template" id="stats-template">
    <% if (done) { %>
        <a id="clear-completed">Clear <%= done %> completed <%= done == 1 ? 'item' : 'items' %></a>
    <% } %>
    <div class="todo-count"><b><%= remaining %></b> <%= remaining == 1 ? 'item' : 'items' %> left</div>
</script>
```

简单的语法，上面的那个对应TodoView。有木有觉得比之前的那一版简洁太多了，有木有！！啥叫代码的美感，对比一下就知道了。

------

首先让我们来回顾一下我们分析的流程：1. 先对页面功能进行了分析；2. 然后又分析了数据模型；3. 最后又对view的功能和代码进行了详解。你是不是觉得这个分析里面少了点什么？没错，就知道经验丰富的你已经看出来了，这里面少了对于流程的分析。这篇文章就对整体流程进行分析。

所以从我的分析中可以看的出来，我是先对各个原材料进行分析，然后再整体的分析（当然前提是我是理解流程的），这并不是分析代码的唯一方法，有时我也会采用跟着流程分析代码的方法。当然还有很多其他的分析方法，大家都有自己的套路嘛。

下面简单的说说流程分析的方法。记得多年前在学vb的时候，分析一个完整项目代码的时候，习惯从程序的入口点开始分析。虽然web网站和桌面软件的实现不同，但是大致思路是一样的（同时也有网站即软件的说法，在RESTful架构中）。所以我们要先找到网站的入口点所在。

和桌面应用项目的分析一样，网站的入口点就在于网页加载的时候。对于todos，自然就是在页面加载完之后执行的操作了，然后就看到下面的代码。

首先是对AppView的一个实例化：

```js
var App = new AppView;
```

实例化，自然就会调用构造函数：

```js
//在初始化过程中，绑定事件到Todos上，
//当任务列表改变时会触发对应的事件。
//最后从localStorage中fetch数据到Todos中。
initialize: function() {
    this.input = this.$("#new-todo");
    this.allCheckbox = this.$("#toggle-all")[0];

    this.listenTo(Todos, 'add', this.addOne);
    this.listenTo(Todos, 'reset', this.addAll);
    this.listenTo(Todos, 'all', this.render);

    this.footer = this.$('footer');
    this.main = $('#main');

    Todos.fetch();
},
```

注意其中的Todos.fetch()方法，前面说过，这个项目是在客户端保存数据，所以使用fetch方法并不会发送请求到服务器。另外在前面关于collection的单独讲解中我们也介绍了fetch执行完成之后，会调用set（默认）或者reset（需要手动设置 {reset: true} ）。所以在没有指明fetch的reset参数的情况下，backbonejs的Collection中的set方法会遍历Todos的内容并且调用add方法。

在initialize中我们绑定了add到addOne上，因此在fetch的时候会Backbonejs会帮我们调用addOne（其实也是在collection的set方法中）。和collection中的set类似的，我们可以自定义reset方法，自行来处理fetch到得数据，但是需要在fetch时手动添加reset参数。


这里先来看下我们绑定到reset上的addAll方法是如何处理fetch过来的数据的:

```js
// 添加一个任务到页面id为todo-list的div/ul中
addOne: function(todo) {
    var view = new TodoView({model: todo});
    this.$("#todo-list").append(view.render().el);
},

// 把Todos中的所有数据渲染到页面,页面加载的时候用到
addAll: function() {
    Todos.each(this.addOne, this);
},
```

在addAll中调用addOne方法，关于Todos.each很好理解，就是语法糖（简化的for循环）。关于addOne方法的细节下面介绍。

然后再来看添加任务的流程，一个良好的代码命名风格始终是让人满心欢喜的。因此很显然，添加一个任务，自然就是addOne,其实你看events中的绑定也能知道，先看一下绑定：

```js
// 绑定dom节点上的事件
events: {
    "keypress #new-todo":  "createOnEnter",
    "click #clear-completed": "clearCompleted",
    "click #toggle-all": "toggleAllComplete"
},
```

这里并没有addOne方法的绑定，但是却有createOnEnter，语意其实一样的。来看主线，createOnEnter这个方法：

```js
/* 创建一个任务的方法，使用backbone.collection的create方法。将数据保存到localStorage,这是一个html5的js库。需要浏览器支持html5才能用。 */
createOnEnter: function(e) {
    if (e.keyCode != 13) return;
    if (!this.input.val()) return;

    //创建一个对象之后会在backbone中动态调用Todos的add方法，该方法已绑定addOne。
    Todos.create({title: this.input.val()});
    this.input.val('');
},
```

注释已写明，Todos.create会调用addOne这个方法。由此顺理成章的来到addOne里面：

```js
//添加一个任务到页面id为todo-list的div/ul中
addOne: function(todo) {
    var view = new TodoView({model: todo});
    this.$("#todo-list").append(view.render().el);
},
```

在里面实例化了一个TodoView类，前面我们说过，这个类是主管各个任务的显示的。具体代码就不细说了。

有了添加再来看更新，关于单个任务的操作，我们直接找TodoView就ok了。所以直接找到

```js
// 为每一个任务条目绑定事件
events: {
    "click .toggle"   : "toggleDone",
    "dblclick .view"  : "edit",
    "click a.destroy" : "clear",
    "keypress .edit"  : "updateOnEnter",
    "blur .edit"      : "close"
},
```

其中的edit事件的绑定就是更新的一个开头，而updateOnEnter就是更新的具体动作。所以只要搞清楚这俩方法的作用一切就明了了。这里同样不用细说。

在往后还有删除一条记录以及清楚已有记录的功能，根据上面的分析过程，我想大家都很容易的去‘顺藤模瓜’。

关于Todos的分析到此就算完成了。

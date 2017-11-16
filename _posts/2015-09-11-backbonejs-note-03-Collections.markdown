---
layout: post
title: Backbone.Collections实践
category: Document
tags: backbone
date: 2015-09-11 11:14:00
published: true
summary: Collections和Model一起使用更精彩
image: pirates.svg
comment: true
latex: false
---

上一节介绍了model的使用，model算是对现实中某一物体的抽象，比如你可以定义一本书的model，具有书名（title）还有书页（page_num)等属性。仅仅用一个Model是不足以呈现现实世界的内容，因此基于Model，这节我们来看collection。collection是model对象的一个有序的集合，也可以理解为是model的容器。概念理解起来十分简单，在通过几个例子来看一下，会觉得更容易理解。

## book和bookshelf的例子

```js
var Book = Backbone.Model.extend({

    defaults : {
        title:'default'
    },

    initialize: function(){
        //alert('Hey, you create me!');
    }

});

var BookShelf = Backbone.Collection.extend({
    model : Book
});

var book1 = new Book({title : 'book1'});
var book2 = new Book({title : 'book2'});
var book3 = new Book({title : 'book3'});

//注意这里面是数组,或者使用add
//var bookShelf = new BookShelf([book1, book2, book3]);

var bookShelf = new BookShelf;

bookShelf.add(book1);
bookShelf.add(book2);
bookShelf.add(book3);
bookShelf.remove(book3);


//基于underscore这个js库，还可以使用each的方法获取collection中的数据

bookShelf.each(function(book){
    alert(book.get('title'));
});
```

很容易理解。

## 使用fetch从服务器端获取数据

首先要在上面的的Bookshelf中定义url，注意collection中并没有urlRoot这个属性。或者你直接在fetch方法中定义url的值，如下：

```js
//注意这里
bookShelf.url = '/books/';
bookShelf.fetch({
    success:function(collection, response, options){
        collection.each(function(book){
            alert(book.get('title'));
        });
    },error:function(collection, response, options){
        alert('error');
    }
});
```

其中也定义了两个接受返回值的方法，具体含义我想很容易理解，返回正确格式(json)的数据，就会调用success方法，错误格式的数据就会调用error方法，当然error方法也看添加和success方法一样的形参。

对应的BookShelf的返回格式如下：[{'title':'book0'},{'title':'book1'}.....]

## reset方法

使用这个方法的时候是要和上面的fetch进行配合的，collection在fetch到数据之后，默认情况会调用set方法(set方法向collection里面添加新model，如果该model之前存在则会合并，与此同时会触发collection的add事件)，可以通过参数{reset: true}来手动触发reset，reset会整个清空collection重新添加所有model。这时你就需要在collection中定义reset方法或者是绑定reset方法。这里使用绑定演示：

```js
var showAllBooks = function(){
    bookShelf.each(function(book){
        //将book数据渲染到页面的操作。
        document.writeln(book.get('title'));
    });
}

bookShelf.bind('reset',showAllBooks);
bookShelf.url = '/books/'; //注意这里
bookShelf.fetch({
    // 需要主动传递reset，才会触发reset
    reset: true,
    success:function(collection, response, options){
        collection.each(function(book){
            alert(book.get('title'));
        });
    },error:function(collection, response, options){
        alert('error');
    }
});
```

_绑定的步骤要在fetch之前进行。_

## 发送数据到Server端

创建数据，其实就是调用collection的create方法，POST对应的Model对象（json数据）到配置好的url上。之后会返回一个model的实例，如下面代码中的onebook。

```js
var NewBooks = Backbone.Collection.extend({
    model: Book,
    url: '/books/'
});

var books = new NewBooks;

var onebook = books.create({
    title: "I'm coming",
});
```

完整代码可以在 code 中找到, 服务器端的代码后面会介绍。


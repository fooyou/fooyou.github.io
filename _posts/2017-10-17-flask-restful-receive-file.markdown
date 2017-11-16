---
layout: post
title: flask-restful 上传文件
category: Document
tags: web
date: 2017-10-17 16:10:44
published: true
summary: flask-restful 如何接受并保存文件
image: pirates.svg
comment: true
---

## 前言

*今天中共十九大，祝贺一下！中国加油！大大们加油！*

![daoni](../img/posts/2017-11-18-liyunlong.jpg)

貌似贴错图了，应该是这个：

![xidad](../img/posts/2017-10-18-xidada.jpg)


## 正文

flask-restful 编写起 restful api 简直是神器（想到李云龙的大刀），只需要做我想做的事就可以了。

话说看了 flask-restful 的文档仍不知怎么读取 POST 的文件，google 了一个[小哥的实现](https://gist.github.com/RishabhVerma/7228939)才恍然大悟

原来要读取文件内容需要重写 `reqparse.Argument` 方法，实现步骤如下：

### Step 1: 实现文件 Argument:

```python
class FileStorageArgument(Argument):
    '''
    此类用于接受处理 flask-restful 收到的所有上传文件
    '''
    def convert(self, value, op):
        if self.type is FileStorage:
            # 如果 Argument 类型为文件，那么返回的 value 则为文件对象
            # 然后在 put 或 post 方法里用这个对象读取文件就好了。
            return value
        super(FileStorageArgument, self).convert(**args, **kwargs)
```

### Step 2: 声明 RequestParser 时指定 argument_class 为文件 Argument

```python
parser = reqparse.RequestParser(argument_class=FileStorageArgument)
```

然后定义参数就可以接受了！

### 完整代码如下：

```python
#!/usr/bin/env python
# coding: utf-8
# @File Name: tspredict.py
# @Author: Joshua Liu
# @Email: liuchaozhenyu@gmail.com
# @Create Date: 2017-10-16 16:10:57
# @Last Modified: 2017-10-18 09:10:08
# @Description:
#   flask 框架实现的 Restful API 接口

from flask import Flask
from flask_restful import Resource
from flask_restful import Api
from flask_restful import reqparse
from flask_restful.reqparse import Argument
from flask_restful import abort
from werkzeug.datastructures import FileStorage
from io import StringIO
import uuid
import csv
import sys

app = Flask(__name__)
api = Api(app)

class FileStorageArgument(Argument):
    '''
    用于 flask-restful 接受所有的上传文件
    '''
    def convert(self, value, op):
        if self.type is FileStorage:
            return value
        super(FileStorageArgument, self).convert(**args, **kwargs)

#
# DataApi
#
class DataApi(Resource):
    '''
    数据接口
    '''
    def __init__(self):
        self.parser = reqparse.RequestParser(argument_class=FileStorageArgument)
        self.parser.add_argument('file', required=True, type=FileStorage, help='csv file', location='files')
        super(DataApi, self).__init__()


    def post(self):
        args = self.parser.parse_args()
        csvfile = args['file']
        fid = self.genid()
        with open(ddir + fid + '.csv', 'wb') as f:
            csvfile.save(f)
		return 'OK'


api.add_resource(DataApi, '/data/')

if __name__ == '__main__':
    app.run(debug=True)
```


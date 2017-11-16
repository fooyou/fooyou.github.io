---
layout: post
title: Flask系列教程之二：Template
category: Document
tags: flask
year: 2015
month: 06
day: 12
published: true
summary: 跟随大师 Miguel Grinberg 的脚步学习Flask，系列教程二——template。
image: pirates.svg
comment: true
---

## 为什么需要模板？

考虑以下需求：在我的主页上显示html的标题，可以这么做：

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
@app.route('/index')
def index():
    user = {'nickname': 'Joshua'} # 占位对象，ugly!
    return '''
<html>
  <head>
    <title>Home Page</title>
  </head>
  <body>
    <h1>Hello, ''' + user['nickname'] + '''</h1>
  </body>
</html>
'''


if __name__ == '__main__':
    app.run()
```

> Refer: http://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-ii-templates

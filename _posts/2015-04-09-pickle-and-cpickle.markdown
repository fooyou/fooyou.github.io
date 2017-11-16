---
layout: post
title: pickle和cPickle：Python对象的序列化
category: Document
tags: python serialization multi-process
year: 2015
month: 04
day: 09
published: true
summary: pickle模块将对象序列化到文件。
image: pirates.svg
comment: true
---

*目的：Python对象序列化*

*可用性：pickle至少1.4版本，cPickle 1.5版本以上*

------

`pickle`模块实现了一种算法，将任意一个Python对象转化成一系列字节（byets）。此过程也调用了`serializing`对象。代表对象的字节流之后可以被传输或存储，再重构后创建一个拥有相同特征（the same characteristics）的新的对象。

`cPickle`使用C而不是Python，实现了相同的算法。这比Python实现要快好几倍，但是它不允许用户从Pickle派生子类。如果子类对你的使用来说无关紧要，那么cPickle是个更好的选择。

*__警告:__ 本文档直接说明，pickle不提供安全保证。如果你在多线程通信（inter-process communication）或者数据存储或存储数据中使用pickle，一定要小心。请勿信任你不能确定为安全的数据。*


## 导入

------

如平常一样，尝试导入cPickle，给它赋予一个别名“pickle”。如果因为某些原因导入失败，退而求其次到Python的原生（native）实现pickle模块。如果cPickle可用，能给你提供一个更快速的执行，否则只能是轻便的执行（the portable implementation）。


```python
try:
   import cPickle as pickle
except:
   import pickle
```


## 编码和解码

------

第一个例子将一种数据结构编码成一个字符串，然后把该字符串打印至控制台。使用一种包含所有原生类型（native types）的数据结构。任何类型的实例都可被腌渍（pickled，译者注：模块名称pickle的中文含义为腌菜），在稍后的例子中会演示。使用pickle.dumps()来创建一个表示该对象值的字符串。


```python
try:
    import cPickle as pickle
except:
    import pickle
import pprint

data = [ { 'a':'A', 'b':2, 'c':3.0 } ]
print 'DATA:',
pprint.pprint(data)

data_string = pickle.dumps(data)
print 'PICKLE:', data_string
```

pickle默认仅由ASCII字符组成。也可以使用更高效的二进制格式（binary format），只是因为在打印的时候更易于理解，本页的所有例子都使用ASCII输出。

```
$ python pickle_string.py

DATA:[{'a': 'A', 'b': 2, 'c': 3.0}]
PICKLE: (lp1
(dp2
S'a'
S'A'
sS'c'
F3
sS'b'
I2
sa.
```

## 重构对象的问题

------

当与你自己的类一起工作时，你必须保证类被腌渍出现在读取pickle的进程的命名空间中。只有该实例的数据而不是类定义被腌渍。类名被用于在反腌渍时，找到构造器（constructor）以创建新对象。以此——往一个文件写入一个类的实例为例：

```python
try:
    import cPickle as pickle
except:
    import pickle
import sys

class SimpleObject(object):

    def __init__(self, name):
        self.name = name
        l = list(name)
        l.reverse()
        self.name_backwards = ''.join(l)
        return

if __name__ == '__main__':
    data = []
    data.append(SimpleObject('pickle'))
    data.append(SimpleObject('cPickle'))
    data.append(SimpleObject('last'))

    try:
        filename = sys.argv[1]
    except IndexError:
        raise RuntimeError('Please specify a filename as an argument to %s' % sys.argv[0])

    out_s = open(filename, 'wb')
    try:
        # 写入流中
        for o in data:
            print 'WRITING: %s (%s)' % (o.name, o.name_backwards)
            pickle.dump(o, out_s)
    finally:
        out_s.close()
```

在运行时，该脚本创建一个以在命令行指定的参数为名的文件：

```
$ python pickle_dump_to_file_1.py test.dat

WRITING: pickle (elkcip)
WRITING: cPickle (elkciPc)
WRITING: last (tsal)
```

一个在读取结果腌渍对象失败的简化尝试：

```python
try:
    import cPickle as pickle
except:
    import pickle
import pprint
from StringIO import StringIO
import sys


try:
    filename = sys.argv[1]
except IndexError:
    raise RuntimeError('Please specify a filename as an argument to %s' % sys.argv[0])

in_s = open(filename, 'rb')
try:
    # 读取数据
    while True:
        try:
            o = pickle.load(in_s)
        except EOFError:
            break
        else:
            print 'READ: %s (%s)' % (o.name, o.name_backwards)
finally:
    in_s.close()
```

该版本失败的原因在于没有 SimpleObject 类可用：

```
$ python pickle_load_from_file_1.py test.dat

Traceback (most recent call last):
  File "pickle_load_from_file_1.py", line 52, in <module>
    o = pickle.load(in_s)
AttributeError: 'module' object has no attribute 'SimpleObject'
```

正确的版本从原脚本中导入 SimpleObject ，可成功运行。

添加：

```python
from pickle_dump_to_file_1 import SimpleObject
```

至导入列表的尾部，接着重新运行该脚本：

```
$ python pickle_load_from_file_2.py test.dat

READ: pickle (elkcip)
READ: cPickle (elkciPc)
READ: last (tsal)
```

当腌渍有值的数据类型不能被腌渍时（套接字、文件句柄（file handles）、数据库连接等之类的），有一些特别的考虑。因为使用值而不能被腌渍的类，可以定义 \_\_getstate\_\_() 和 \_\_setstate\_\_() 来返回状态（state）的一个子集，才能被腌渍。新式类（New-style classes）也可以定义\_\_getnewargs\_\_()，该函数应当返回被传递至类内存分配器（the class memory allocator）（C.\_\_new\_\_()）的参数。使用这些新特性的更多细节，包含在标准库文档中。


## 环形引用（Circular References）

------

pickle协议（pickle protocol）自动处理对象间的环形引用，因此，即使是很复杂的对象，你也不用特别为此做什么。考虑下面这个图：

<!-- Title: complex Pages: 1 -->
<svg width="98pt" height="260pt"
viewBox="0.00 0.00 98.00 260.00" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
<g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 256)">
<title>complex</title>
<!-- root -->
<g id="node1" class="node"><title>root</title>
<ellipse fill="none" stroke="#dddddd" cx="63" cy="-234" rx="27" ry="18"/>
<text text-anchor="middle" x="63" y="-230.3" font-family="Times,serif" font-size="14.00" fill="#dddddd">root</text>
</g>
<!-- a -->
<g id="node2" class="node"><title>a</title>
<ellipse fill="none" stroke="#dddddd" cx="27" cy="-162" rx="27" ry="18"/>
<text text-anchor="middle" x="27" y="-158.3" font-family="Times,serif" font-size="14.00" fill="#dddddd">a</text>
</g>
<!-- root&#45;&gt;a -->
<g id="edge1" class="edge"><title>root&#45;&gt;a</title>
<path fill="none" stroke="#dddddd" d="M54.6504,-216.765C50.2885,-208.283 44.8531,-197.714 39.9587,-188.197"/>
<polygon fill="#dddddd" stroke="#dddddd" points="42.9904,-186.439 35.3043,-179.147 36.7654,-189.641 42.9904,-186.439"/>
</g>
<!-- b -->
<g id="node3" class="node"><title>b</title>
<ellipse fill="none" stroke="#dddddd" cx="27" cy="-90" rx="27" ry="18"/>
<text text-anchor="middle" x="27" y="-86.3" font-family="Times,serif" font-size="14.00" fill="#dddddd">b</text>
</g>
<!-- root&#45;&gt;b -->
<g id="edge2" class="edge"><title>root&#45;&gt;b</title>
<path fill="none" stroke="#dddddd" d="M71.865,-216.681C80.5522,-198.388 91.1272,-168.13 81,-144 75.2881,-130.39 64.1878,-118.518 53.5293,-109.479"/>
<polygon fill="#dddddd" stroke="#dddddd" points="55.5654,-106.626 45.5615,-103.137 51.2063,-112.103 55.5654,-106.626"/>
</g>
<!-- a&#45;&gt;a -->
<g id="edge4" class="edge"><title>a&#45;&gt;a</title>
<path fill="none" stroke="#dddddd" d="M46.895,-174.432C59.688,-177.675 72,-173.531 72,-162 72,-153.622 65.5006,-149.143 57.0395,-148.564"/>
<polygon fill="#dddddd" stroke="#dddddd" points="56.5019,-145.1 46.895,-149.568 57.191,-152.066 56.5019,-145.1"/>
</g>
<!-- a&#45;&gt;b -->
<g id="edge3" class="edge"><title>a&#45;&gt;b</title>
<path fill="none" stroke="#dddddd" d="M21.1601,-144.411C20.2975,-136.507 20.0481,-126.852 20.4119,-117.935"/>
<polygon fill="#dddddd" stroke="#dddddd" points="23.9033,-118.179 21.1206,-107.956 16.9209,-117.683 23.9033,-118.179"/>
</g>
<!-- b&#45;&gt;a -->
<g id="edge6" class="edge"><title>b&#45;&gt;a</title>
<path fill="none" stroke="#dddddd" d="M32.8794,-107.956C33.7139,-115.827 33.9485,-125.374 33.5831,-134.187"/>
<polygon fill="#dddddd" stroke="#dddddd" points="30.0742,-134.184 32.8399,-144.411 37.0558,-134.691 30.0742,-134.184"/>
</g>
<!-- c -->
<g id="node4" class="node"><title>c</title>
<ellipse fill="none" stroke="#dddddd" cx="27" cy="-18" rx="27" ry="18"/>
<text text-anchor="middle" x="27" y="-14.3" font-family="Times,serif" font-size="14.00" fill="#dddddd">c</text>
</g>
<!-- b&#45;&gt;c -->
<g id="edge5" class="edge"><title>b&#45;&gt;c</title>
<path fill="none" stroke="#dddddd" d="M27,-71.6966C27,-63.9827 27,-54.7125 27,-46.1124"/>
<polygon fill="#dddddd" stroke="#dddddd" points="30.5001,-46.1043 27,-36.1043 23.5001,-46.1044 30.5001,-46.1043"/>
</g>
</g>
</svg>

上图虽然包括几个环形引用，但也能以正确的结构腌渍和重新读取（reloaded）。

```python
import pickle

class Node(object):
    """
    一个所有结点都可知它所连通的其它结点的简单有向图。
    """
    def __init__(self, name):
        self.name = name
        self.connections = []
        return

    def add_edge(self, node):
        "创建两个结点之间的一条边。"
        self.connections.append(node)
        return

    def __iter__(self):
        return iter(self.connections)

def preorder_traversal(root, seen=None, parent=None):
    """产生器（Generator ）函数通过一个先根遍历（preorder traversal）生成（yield）边。"""
    if seen is None:
        seen = set()
    yield (parent, root)
    if root in seen:
        return
    seen.add(root)
    for node in root:
        for (parent, subnode) in preorder_traversal(node, seen, root):
            yield (parent, subnode)
    return

def show_edges(root):
    "打印图中的所有边。"
    for parent, child in preorder_traversal(root):
        if not parent:
            continue
        print '%5s -> %2s (%s)' % (parent.name, child.name, id(child))

# 创建结点。
root = Node('root')
a = Node('a')
b = Node('b')
c = Node('c')

# 添加边。
root.add_edge(a)
root.add_edge(b)
a.add_edge(b)
b.add_edge(a)
b.add_edge(c)
a.add_edge(a)

print 'ORIGINAL GRAPH:'
show_edges(root)

# 腌渍和反腌渍该图来创建
# 一个结点集合。
dumped = pickle.dumps(root)
reloaded = pickle.loads(dumped)

print
print 'RELOADED GRAPH:'
show_edges(reloaded)
```

重新读取的诸多节点（译者注：对应图中的圆圈）不再是同一个对象，但是节点间的关系保持住了，而且读取的仅仅是带有多个引用的对象的一个拷贝。上面所说的可以通过测试各节点在pickle处理前和之后的id()值来验证。

```
$ python pickle_cycle.py

ORIGINAL GRAPH:
 root ->  a (4299721744)
    a ->  b (4299721808)
    b ->  a (4299721744)
    b ->  c (4299721872)
    a ->  a (4299721744)
 root ->  b (4299721808)

RELOADED GRAPH:
 root ->  a (4299722000)
    a ->  b (4299722064)
    b ->  a (4299722000)
    b ->  c (4299722128)
    a ->  a (4299722000)
 root ->  b (4299722064)
```

> 本文参见：http://segmentfault.com/a/1190000002493548 和英文原文:http://pymotw.com/2/pickle/
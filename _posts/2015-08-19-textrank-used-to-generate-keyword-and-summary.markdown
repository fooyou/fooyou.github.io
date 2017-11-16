---
layout: post
title: 使用TextRank算法为文本生成关键字和摘要
category: Document
tags: algorithm
date: 2015-08-19 15:23:43
published: true
summary: 使用TextRank算法为文本生成关键字和摘要。其比TF-IDF效果要好，因为TF-IDF依赖于语料词库，而TextRank对其依赖较小，但不论如何现在的算法不可能像人那样理解语言所要表达的信息，它只是用机器可理解的方式对文本句子和词进行权重计算而已。
image: pirates.svg
comment: true
latex: true
---

TextRank算法基于PageRank，使用TextRank算法为文本生成关键字和摘要。其比TF-IDF效果要好，因为TF-IDF依赖于语料词库，要训练一个能覆盖全互联网或者是某个领域的全语料是非常难的，更何况每天网上的新内容都大的惊人。而TextRank对其依赖较小，所以用起来更方便，效果也不错。但不论如何现在的算法不可能像人那样理解语言所要表达的信息，它只是用机器可理解的方式对文本句子和词进行权重计算而已。

## PageRank

PageRank通过网络浩瀚的超链接关系来确定一个页面的等级。Google把从A页面到B页面的链接解释为A页面给B页面的投票。一个高等级的页面可以提升其相邻页面的等级。一个页面的PageRank是由所有链向它的页面的重要性经过递归算法得到的。

PageRank论文如下：

> [The PageRank Citation Ranking: Bringing Order to the Web](http://ilpubs.stanford.edu:8090/422/1/1999-66.pdf)

_PageRank的专利权属于斯坦福大学_

### PageRank算法

#### 简单的

假设一个由4个页面组成的小团体：A，B，C，D。如果所有页面都链接到A，如下图示：

```
{{ digraph }}
rankdir="TD"
bgcolor="transparent";
node [color="#dddddd" fontcolor="#dddddd"]
edge [color="#dddddd"]
B -> A;
C -> A;
D -> A;
{{ enddigraph }}
```

_图1:_

那么A的PR（PageRank）将是BCD的PR总和：

$$ PR(A) = PR(B) + PR(C) + PR(D) $$

继续假设B也有链接到C，并且D也有链接到包括A的3个页面。如下图示：

```
{{ digraph }}
bgcolor="transparent";
node [color="#dddddd" fontcolor="#dddddd"]
edge [color="#dddddd"]
rankdir="LR"
B -> A;
B -> C;
C -> A;
D -> A;
D -> B;
D -> C;
{{ enddigraph }}
```

_图2:_

一个页面不能投票2次，所以B给每个页面半票。以同样逻辑，D投的票只有1/3算到A的PageRank上：

$$ PR(A) = \frac{PR(B)}{2} + \frac{PR(C)}{1} + \frac{PR(D)}{3} $$

也就是说，根据链出总数平分一个页面的PR值：

$$ PR(A) = \frac{PR(B)}{L(B)} + \frac{PR(C)}{L(C)} + \frac{PR(D)}{L(D)} $$

最后，所有这些被换算为一个百分比再乘上一个系数d。由于“没有向外链接的页面”传递出去的PageRank会是0，所以，Google通过数学公式给每个页面一个最小值\\(\frac{(1 - d)}{N}\\)：

$$ PR(A) = \left( \frac{PR(B)}{L(B)} + \frac{PR(C)}{LC()} + \frac{PR(D)}{L(D)} + ... \right) d + \frac{1 - d}{N} $$

如果给每个页面一个随机PageRank值（非0），那么经过不断的重复计算，这些页面的PR值会趋向稳定，也就是收敛状态。这就是搜索引擎使用它的原因。

#### 完整的

这个方程式引入了随机浏览的概念，即有人上网无聊随机打开一些页面，点一些链接。一个页面的PageRank值也影响了它被随机浏览的概率。为了便于理解，这里假设上网者不断点网页上的链接，最终到了一个没有任何链出页面的网页，这时候上网者会随机到另外的网页开始浏览。

为了处理那些“没有向外链接的页面”（这些页面就像“黑洞”会吞噬掉用户继续向下浏览的概率）带来的问题，\\( d = 0.85 \\)，（这里的d被称为阻尼系数（damping factor）），其意义是，在任何时刻，用户到达某页面后并继续向后浏览的概率，该数值是根据上网使用浏览器书签平均频率估算而得。\\( 1 - d = 0.15 \\)（就是用户停止点击，随机跳到新URL的概率）的算法被用到了所有页面上。

所以，这个公式如下：

$$ PageRank(p_i) = \frac{1 - d}{N} + d \sum_{p_j \in M(p_i)} \frac{PageRank(p_j)}{L(p_j)} $$

\\( p_1,p_2,...,p_N \\)是被研究的页面，\\(M(p_i)\\)是链入\\(p_i\\)页面的集合，\\(L(p_j)\\)是\\(p_j\\)链出页面的数量，而N是所有页面的数量。

__PageRank__值是一个特殊矩阵中的特征向量。这个特征向量为：

$$
\mathbf{R} =
\begin{bmatrix}
{\rm PageRank}(p_1) \\\
{\rm PageRank}(p_2) \\\
\vdots \\\
{\rm PageRank}(p_N)
\end{bmatrix}
$$

R是等式的答案：

$$
\mathbf{R} =
\begin{bmatrix}
{(1-d) / N} \\\
{(1-d) / N} \\\
\vdots \\\
{(1-d) / N}
\end{bmatrix}
+ d
\begin{bmatrix}
\ell(p_1,p_1) & \ell(p_1,p_2) & \cdots & \ell(p_1,p_N) \\\
\ell(p_2,p_1) & \ddots & & \\\
\vdots & & \ell(p_i,p_j) & \\\
\ell(p_N,p_1) & & & \ell(p_N,p_N)
\end{bmatrix}
\mathbf{R}
$$

如果\\(p_j\\)不链向\\(p_i\\)，而且每个j都成立时，\\( \ell(p_i,p_j) \\)等于0。

$$ \sum_{i = 1}^N \ell(p_i,p_j) = 1 $$

这项技术的主要缺点是旧的页面等级会比新页面高。因为即使是非常好的新页面也不会有很多外链，除非它是某个站点的子站点。

这就是PageRank需要多项算法结合的原因。PageRank似乎偏好于维基百科页面，在条目名称的搜索结果中，维基百科页面总在大多数或者其他所有页面之前。原因主要是维基百科内相互的链接很多，并且有很多站点链入。

Google经常处罚恶意提高PageRank的行为，至于其如何区分正常的链接和不正常的链接仍然是个商业机密。但是在Google的链接规范中，已经很清楚地说明，哪些做法是属于违反指南的行为。

#### 计算图1,2的PR值

__图1__

建立矩阵，两个节点有链接其值为1

   | A | B | C | D 
---|---|---|---|----
 A | 0 | 1 | 1 | 1 
 B | 0 | 0 | 0 | 0
 C | 0 | 0 | 0 | 0
 D | 0 | 0 | 0 | 0

```python
M = [[0, 1, 1, 1], [0, 0, 0, 0], [0, 0, 0, 0], [0, 0, 0, 0]]
```

假定每个网页的PR初始值为1

```python
PR = [[1], [1], [1], [1]]
```

用于计算PageRank的Python代码：

```python
import numpy as np  # 使用numpy进行矩阵计算

M = np.matrix([[0, 1, 1, 1], [0, 0, 0, 0], [0, 0, 0, 0], [0, 0, 0, 0]])
PR = np.matrix([[1], [1], [1], [1]])

for i in range(10):
    PR = 0.15 + 0.85 * M * PR
    print(i)
    print(PR)
```

结果如下：

```
0
[[ 2.7 ]
 [ 0.15]
 [ 0.15]
 [ 0.15]]
1
[[ 0.5325]
 [ 0.15  ]
 [ 0.15  ]
 [ 0.15  ]]

...

9
[[ 0.5325]
 [ 0.15  ]
 [ 0.15  ]
 [ 0.15  ]]
```

可以看出网页A的PageRank是最高的。

__图2__

建立矩阵，一个网页的总值为1，如果有多个链接则均摊。

   | A | B | C | D 
---|---|---|---|----
 A | 0 | .5| 1 | .33 
 B | 0 | 0 | 0 | .33
 C | 0 | .5| 0 | .33
 D | 0 | 0 | 0 | 0

用于计算PageRank的Python代码：

```python
import numpy as np

M = np.matrix([[0, .5, 1, .33], [0, 0, 0, .33], [0, .5, 0, .33], [0, 0, 0, 0]])
PR = np.matrix([[1], [1], [1], [1]])

for i in range(10):
    PR = 0.15 + 0.85 * M * PR
    print(i)
    print(PR)
```

结果如下：

```
0
[[ 1.7055]
 [ 0.4305]
 [ 0.8555]
 [ 0.15  ]]
1
[[ 1.1022125]
 [ 0.192075 ]
 [ 0.3750375]
 [ 0.15     ]]
...
9
[[ 0.50635772]
 [ 0.192075  ]
 [ 0.27370688]
 [ 0.15      ]]
```

可以看到各个网页的PR值的变化。

## TextRank

TextRank是在Google的PageRank算法启发下，针对文本里的句子设计的权重算法，目标是自动摘要。它利用PageRank的投票原理，让每一个单词给它的邻居（TextRank术语称为“窗口”）投赞成票，票的权重取决于自己的票数。像PageRank一样TextRank也采用矩阵迭代收敛的方式解决权重和票数相互依赖的“悖论”。

TextRank 论文如下：

> [TextRank: Bringing Order into Texts](http://web.eecs.umich.edu/~mihalcea/papers/mihalcea.emnlp04.pdf) _Rada Mihalcea and Paul Tarau_

其中对PageRank的表述：

$$ S(V_i) = (1 - d) + d * \sum_{j \in I_n(V_i)} \frac{1}{\left| O_{ut}(V_j) \right|} S(V_j) $$

其中：

- \\( S(V_i) \\)：指第i个网页的PageRank。
- d：阻尼系数，一般为0.85。
- \\( I_n(V_i) \\)：指指向第i个网页的网页集合。
- \\( O_{ut}(V_j) \\)：指第j个网页所链出的网页集合。

__TextRank的公式：__

TextRank算法计算自动摘要的公式：

$$ WS(V_i) = (1 - d) + d * \sum_{V_j \in I_n(V_i)} \frac{w_{ji}}{\sum_{V_k \in O_{ut}(V_j)} w_{jk}} WS(V_j) $$

WS：是weight_sum（权重）的缩写，右侧的求和表示每个相邻句子的贡献程度。

其中： \\( w_{ji} \\)：表示两个句子的相似度。

而关于相似度的计算，有许多方法，比如基于向量空间模型的欧式距离和余弦相似度和基于Hash算法的minhash和simhash等等。论文中给出的相似度计算公式如下：

$$ Similarity(S_i, S_j) = \frac{\left| { w_k | w_k \in S_i  w_k \in S_j } \right|}{log\left(\left| S_i \right|\right) + log\left(\left| S_j \right|\right))} $$

$$ & $$

正规的TextRank公式在PageRank的公式基础上，引入了边的权值概念，代表两个句子的相似度：


> 参考：https://zh.wikipedia.org/wiki/PageRank

> 参考：http://www.hankcs.com/category/nlp/

> 参考：http://my.oschina.net/letiantian/blog/351154




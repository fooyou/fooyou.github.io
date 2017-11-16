---
layout: post
title: 隐马尔可夫模型的基本概念
category: Document
tags: 
date: 2017-03-29 12:03:20
published: true
summary: 
image: pirates.svg
comment: true
---

### 隐马尔可夫模型的定义

隐马尔可夫模型是关于时序的概率模型，描述由一个隐藏的马尔可夫链随机生成不可观测的状态随机序列，再由各个状态生成一个观测而产生观测随机序列的过程。隐藏的马尔可夫链随机生成的状态的序列，称为状态序列（state sequence);每个状态生成一个观测，而由此产生的观测的随机序列，称为观测序列（observation sequence)。序列的每一个位置又可以看作是一个时刻。

什么叫隐马尔科夫链呢？简单说来，就是把时间线看做一条链，每个节点只取决于前N个节点。就像你打开朋友圈，发现你可以根据你的基友最近的几条状态猜测出接下来TA狗嘴里要吐什么东西一样。

接下来引入一些符号来表述这个定义——

设Q是所有可能的状态的集合，V是所有可能的观测的集合。

$$Q = \{q_1,q_2,\cdots,q_N\}, V = \{v_1, v_2,\cdots,v_M\}$$

其中，**N**是可能的状态数，**M**是可能的观测数。

状态 **q** 是不可见的，观测 **v** 是可见的。应用到词性标注系统，词就是 **v**，词性就是 **q**。

**I** 是长度为 **T** 的状态序列，**O** 是对应的观测序列。

$$I = (i_1,i_2,\cdots,i_T), O = (o_1,o_2,\cdots,o_T)$$

这可以想象为相当于给定了一个词（O）+词性（I）的训练集，于是我们手上有了一个可以用隐马尔可夫模型解决的实际问题。

定义 **A** 为状态转移概率矩阵：

$$ A = \left[a_{ij}\right]_{N \times N}
$$

其中，

$$a_{ij} = P(i_{i+1} = q_j | i_t = q_i); i,j \in (1,2,\cdots,N)$$

是在时刻 $t$ 处于状态 $q_i$ 的条件下在时刻 $t+1$ 转移到状态 $q_j$ 的概率。

这实际在表述一个一阶的HMM，所作的假设是每个状态只跟前一个状态有关。

B是观测概率矩阵: $$B=\left[b_j(k) \right]_{N \times M}$$

其中，$$b_j(k) = P(o_t=v_k|i_t=q_j), k=1,2,\cdots,M; j=1,2,\cdots,N$$

是在时刻 $t$ 处于状态 $q_j$ 的条件下生成观测 $v_k$ 的概率（也就是所谓的“发射概率”）。

这实际上在作另一个假设，观测是由当前时刻的状态决定的，跟其他因素无关，这有点像Moore自动机。

$\pi$是初始状态概率向量：$$\pi=(\pi_i)$$，其中，$$\pi_i=P(i_1=q_i), i=1,2,\cdots,N$$

是时刻 $t=1$ 处于状态 $q_j$ 的概率。

隐马尔可夫模型由初始状态概率向量 $\pi$、状态转移概率矩阵 $A$ 和观测概率矩阵 $B$ 决定。$\pi$和 $A$ 决定状态序列，$B$ 决定观测序列。因此，隐马尔可夫模型 $\lambda$ 可以用三元符号表示，即 $$\lambda = (A,B,\pi)$$

$A,B,\pi$ 称为隐马尔可夫模型的三要素。如果加上一个具体的状态集合Q和观测序列V，则构成了HMM的五元组，又是一个很有中国特色的名字吧。

状态转移概率矩阵 $A$ 与初始状态概率向量 $π$ 确定了隐藏的马尔可夫链，生成不可观测的状态序列。观测概率矩阵 $B$ 确定了如何从状态生成观测，与状态序列综合确定了如何产生观测序列。

从定义可知，隐马尔可夫模型作了两个基本假设：

(1): 齐次马尔可夫性假设，即假设隐藏的马尔可夫链在任意时刻t的状态只依赖于其前一时刻的状态，与其他时刻的状态及观测无关。

$$P(i_t|i_{t-1}, o_{t-1}, \cdots, i_1, o_1)=P(i_t|i_{t-1}), t = 1,2,\cdots,T$$

从上式左右两边的复杂程度来看，齐次马尔可夫性假设简化了许多计算。

(2): 观测独立性假设，即假设任意时刻的观测只依赖于该时刻的马尔可夫链的状态，与其他观测及状态无关。

$$P(o_t|i_T,o_T,i_{T-1},o_{T-1},\cdots,i_{t+1},o_{t+1},i_t,o_t,i_{t-1},o_{t-1},\cdots,i_1,o_1)=P(o_t|i_t)$$

依然是一个简化运算的假设。

### 隐马尔可夫模型的实例

让我们抛开教材上拗口的红球白球与盒子模型吧，来看看这样一个来自[wiki](https://en.wikipedia.org/wiki/Viterbi_algorithm#Example)的经典的HMM例子：

你本是一个城乡结合部修电脑做网站的小码农，突然春风吹来全民创业。于是跟村头诊所的老王响应总理号召合伙创业去了，有什么好的创业点子呢？对了，现在不是很流行什么“大数据”么，那就搞个“医疗大数据”吧，虽然只是个乡镇诊所……但管它呢，投资人就好这口。

数据从哪儿来呢？你把老王，哦不，是王老板的出诊记录都翻了一遍，发现从这些记录来看，村子里的人只有两种病情：要么健康，要么发烧。但村民不确定自己到底是哪种状态，只能回答你感觉正常、头晕或冷。有位村民是诊所的常客，他的病历卡上完整地记录了这三天的身体特征(正常、头晕或冷)，他想利用王老板的“医疗大数据”得出这三天的诊断结果(健康或发烧)。

这时候王老板掐指一算，说其实感冒这种病，只跟病人前一天的病情有关，并且当天的病情决定当天的身体感觉。

于是你一拍大腿，天助我也，隐马尔可夫模型的两个基本假设都满足了，于是统计了一下病历卡上的数据，撸了这么一串Python代码：

```py
states = ('Healthy', 'Fever')
 
observations = ('normal', 'cold', 'dizzy')
 
start_probability = {'Healthy': 0.6, 'Fever': 0.4}
 
transition_probability = {
    'Healthy': {'Healthy': 0.7, 'Fever': 0.3},
    'Fever': {'Healthy': 0.4, 'Fever': 0.6},
}
 
emission_probability = {
    'Healthy': {'normal': 0.5, 'cold': 0.4, 'dizzy': 0.1},
    'Fever': {'normal': 0.1, 'cold': 0.3, 'dizzy': 0.6},
}
```

![HMM示例图](https://upload.wikimedia.org/wikipedia/commons/0/0c/An_example_of_HMM.png)

states代表病情，observations表示最近三天观察到的身体感受，start_probability代表病情的分布，transition_probability是病情到病情的转移概率，emission_probability则是病情表现出身体状况的发射概率。隐马的五元组都齐了，就差哪位老总投个几百万了。

为了方便对照，降低公式造成的眩晕和昏睡效果，我决定改变以前的风格，不再把代码放到最后，而是直接将代码嵌入到理论讲解中，形成一份programmatic的tutorial。本文的代码主要参考 aehuynh 的Python实现，我在其基础上添加了simulate方法和一些utility，并且修改了他错误的Baum-Welch算法，开源在GitHub上：https://github.com/hankcs/hidden-markov-model 。

打开Python编辑器，有了前面的知识，我们就可以动手写一个一阶HMM模型的定义了：

```py
class HMM:
    """
    Order 1 Hidden Markov Model
 
    Attributes
    ----------
    A : numpy.ndarray
        State transition probability matrix
    B: numpy.ndarray
        Output emission probability matrix with shape(N, number of output types)
    pi: numpy.ndarray
        Initial state probablity vector
    """
 
    def __init__(self, A, B, pi):
        self.A = A
        self.B = B
        self.pi = pi
```

为了让该HMM实现接受王总的数据，我们必须写一些utility方法：

```py
def generate_index_map(lables):
    index_label = {}
    label_index = {}
    i = 0
    for l in lables:
        index_label[i] = l
        label_index[l] = i
        i += 1
    return label_index, index_label
 
 
states_label_index, states_index_label = generate_index_map(states)
observations_label_index, observations_index_label = generate_index_map(observations)
 
 
def convert_observations_to_index(observations, label_index):
    list = []
    for o in observations:
        list.append(label_index[o])
    return list
 
 
def convert_map_to_vector(map, label_index):
    v = np.empty(len(map), dtype=float)
    for e in map:
        v[label_index[e]] = map[e]
    return v
 
 
def convert_map_to_matrix(map, label_index1, label_index2):
    m = np.empty((len(label_index1), len(label_index2)), dtype=float)
    for line in map:
        for col in map[line]:
            m[label_index1[line]][label_index2[col]] = map[line][col]
    return m
 
 
A = convert_map_to_matrix(transition_probability, states_label_index, states_label_index)
print A
B = convert_map_to_matrix(emission_probability, states_label_index, observations_label_index)
print B
observations_index = convert_observations_to_index(observations, observations_label_index)
pi = convert_map_to_vector(start_probability, states_label_index)
print pi
 
h = hmm.HMM(A, B, pi)
```

这些方法主要将源数据的map形式转为NumPy的矩阵形式：

```
[[ 0.7  0.3]
 [ 0.4  0.6]]
 
[[ 0.5  0.4  0.1]
 [ 0.1  0.3  0.6]]
 
[ 0.6  0.4]
```

分别对应着 $A$，$B$ 和 $\pi$。虽然该class目前什么都不能做，但是很快我们就能让它派上实际用场。

### 观测序列的生成过程

根据隐马尔可夫模型定义，可以将一个长度为T的观测序列 $O=(o_1,o_2,\cdots,o_T)$ 的生成过程描述如下：

#### 算法——观测序列的生成

输入：隐马尔可夫模型 $\lambda=(A,B,\pi)$，观测序列长度；

输出：观测序列 $O=(o_1,o_2,\cdots,o_T)$。

1. 按照初始状态分布 $\pi$ 产生状态 $i_1$；
2. 令 $t=1$；
3. 按照状态$i_1$的观测概率分布 $b_{i_t}(k)$ 生成 $o_t$；
4. 按照状态 $i_1$ 的状态转移概率分布 $\left\{ a_{i_t i_{t+1}} \right\}$ 产生状态 $i_{t+1},i_{t+2} = 1,2,\cdots,N$

令 $t=t+1$；如果 $t<T$ 则转步 3；否则，终止。

### 观测序列生成Python实现

为什么要生成观测序列呢？虽然我很想剧透并没有什么卵用，但王老板说他基于三年乡镇诊所病历卡的“医疗大数据库”里面并没有足够的数据给你跑什么算法。你只好自己动手，写算法生成一些了：

```py
def simulate(self, T):
 
    def draw_from(probs):
        return np.where(np.random.multinomial(1,probs) == 1)[0][0]
 
    observations = np.zeros(T, dtype=int)
    states = np.zeros(T, dtype=int)
    states[0] = draw_from(self.pi)
    observations[0] = draw_from(self.B[states[0],:])
    for t in range(1, T):
        states[t] = draw_from(self.A[states[t-1],:])
        observations[t] = draw_from(self.B[states[t],:])
    return observations,states
```

这串代码很小巧玲珑，draw_from接受一个概率分布，然后生成该分布下的一个样本。算法首先初始化两个长度为T的向量，接着按照初始状态分布pi生成第一个状态：

```py
states[0] = draw_from(self.pi)
```

这还没完，有了状态，我们还可以取出状态对应的观测的概率分布，生成一个观测：

```py
observations[0] = draw_from(self.B[states[0],:])
```

接下来一直到t，我们都是按前一个状态取出状态转移概率分布，生成状态，再取出状态对应的观测的概率分布，生成一个观测。重复这个步骤，就得到了长度为T的观测和状态向量了。

具体的调用方法是：

```py
observations_data, states_data = h.simulate(10)
print observations_data
print states_data
```

输出：

```py
[1 0 0 1 0 1 0 2 1 2]
[0 0 0 0 0 0 0 0 1 0]
```

这里直接用了下标表示，如果要糊弄投资人的话，还请按照index_label_map转换为相应的label。

### 隐马尔可夫模型的3个基本问題

1. 概率计算问题。给定模型 $\lambda=(A,b,\pi)$ 和观测序列 $O=(o_1,o_2,\cdots,o_T)$，计算在模型 $\lambda$ 下观测序列 $O$ 出现的概率。
2. 学习问题。已知观测序列 $O=(o_1,o_2,\cdots,o_T)$，估计模型 $\lambda=(A,b,\pi)$ 参数，使得在该模型下观测序列概率 $P(O|\lambda)$ 最大。即用极大似然估计得方法估计参数。
3. 预测问题，也称为解码问题。已知模型 $\lambda=(A,b,\pi)$ 和观测序列 $O=(o_1,o_2,\cdots,o_T)$，求对给定观测序列条件概率 $P(I|O)$ 最大的状态序列 $I=(i_1,i_2,\cdots,i_T)$。即给定观测序列，求最有可能的对应的状态序列。

下面各节将逐一介绍这些基本问题的解法。

## 概率计算算法

这节介绍计算观测序列概率的前向（forward)与后向（backward)算法，以及概念上可行但计算上不可行的直接计算法（枚举）。前向后向算法无非就是求第一个状态的前向概率或最后一个状态的后向概率，然后向后或向前递推即可。


### 直接计算法

给定模型，求给定长度为 $T$ 的观测序列的概率，直接计算法的思路是枚举所有的长度 $T$ 的状态序列，计算该状态序列与观测序列的联合概率（隐状态发射到观测），对所有的枚举项求和即可。在状态种类为 $N$ 的情况下，一共有$ N^T$ 种排列组合，每种组合计算联合概率的计算量为 $T$，总的复杂度为 $O(TN^T)$，并不可取。

### 前向算法

首先定义前向概率。

定义（前向概率）：给定隐马尔可夫模型 $\lambda$，定义到时刻 $t$ 为止的观测序列为 $o_1,o_2,\cdots,o_t$ 且状态为 $q_i$ 的概率为前向概率，记作 

$$\alpha(i)=P(o_1,o_2,\cdots,o_t,i_t=q_i|\lambda)$$

可以递推地求得前向概率$\alpha_t(i)$及观测序列概率$P(O|\lambda)$。

#### 观测序列概率的前向算法

输入：隐马尔可夫模型 $\lambda$，观测序列 $O$；

输出：观测序列概率 $P(O|\lambda)$；

步骤：

1. 初值 $$\alpha_1(i)=\pi_ib_i(o_1), i=1,2,\cdots,N$$ 前向概率的定义中一共限定了两个条件，一是到当前为止的观测序列，另一个是当前的状态。所以初值的计算也有两项（观测和状态），一项是初始状态概率，另一项是发射到当前观测的概率。

2. 递推对 $t=1,2,\cdots,T-1,$ $$\alpha_{t+1}(i)=\left[\sum_{j=1}^N \alpha_t(j)\alpha_{ji} \right]b_i(o_{t+1}), i=1,2,\cdots,N$$ 每次递推同样由两部分构成，大括号中是当前状态为i且观测序列的前t个符合要求的概率，括号外的是状态i发射观测 t+1的概率。

3. 终止 $$P(O|\lambda)=\sum_{i=1}^N \alpha_T(i)$$ 由于到了时间T，一共有N种状态发射了最后那个观测，所以最终的结果要将这些概率加起来。

由于每次递推都是在前一次的基础上进行的，所以降低了复杂度。准确来说，其计算过程如下图所示：

![](https://github.com/fooyou/fooyou.github.io/blob/master/img/dots/pre_direction.dot.png?raw=true)

下方标号表示时间节点，每个时间点都有N种状态，所以相邻两个时间之间的递推消耗 $N^2$ 次计算。而每次递推都是在前一次的基础上做的，所以只需累加 $O(T)$ 次，所以总体复杂度是 $O(T)$ 个 $N^2$ ，即 $O(N^2T)$。

#### 前向算法Python实现

```py
def _forward(self, obs_seq):
    N = self.A.shape[0]
    T = len(obs_seq)
 
    F = np.zeros((N,T))
    F[:,0] = self.pi * self.B[:, obs_seq[0]]
 
    for t in range(1, T):
        for n in range(N):
            F[n,t] = np.dot(F[:,t-1], (self.A[:,n])) * self.B[n, obs_seq[t]]
 
    return F
```

代码和伪码的对应关系还是很清晰的，F对应alpha，HMM的三个参数与伪码一致。

### 后向算法

定义（后向概率）：给定隐马尔可夫模型 $\lambda$，定义在时刻 $t$ 状态为 $q_i$ 的条件下，从 $t+1$ 到 $T$ 的部分观测序列为 $o_{t+1},o_{t+2},\cdots,o_T$ 的概率为后向概率。记作 $$\beta_t(i)=P(o_{t+1},o_{t+2},\cdots,o_T|i_t=q_i, \lambda)$$

可以用递推的方法求得后向概率 $\beta_t(i)$ 及观测序列概率 $P(O|\lambda)$。

#### 观测序列概率的后向算法

输入：隐马尔可夫模型 $\lambda$，观测序列 $O$；

输出：观测序列概率 $P(O|\lambda)$；

步骤：

1. 初值 $$\beta_T(i) = 1, i=1,2,\cdots,N$$ 根据定义，从 $T+1$ 到 $T$ 的部分观测序列其实不存在，所以硬性规定这个值是1。
2. 对 $t=T-1,T-2,\cdots,1$ $$\beta_t(i)=\sum_{j=1}^N a_{ij}b_j(o_{t+1}) \beta_{t+1}(j), i=1,2,\cdots,N$$  $a_{ij}$ 表示状态i转移到j的概率，$b_j$ 表示发射 $O_{t+1}$，$\beta_{t+1}(j)$ 表示j后面的序列对应的后向概率。
3. $$P(O|\lambda)=\sum_{i=1}^N \pi_ib_i(o1)\beta_1(i)$$ 最后的求和是因为，在第一时间点上有 $N$ 种后向概率都能输出从2到T的观测序列，所以乘上输出 $O_1$ 的概率后求和得到最终结果。

#### 后向算法的Python实现

```py
def _backward(self, obs_seq):
    N = self.A.shape[0]
    T = len(obs_seq)
 
    X = np.zeros((N,T))
    X[:,-1:] = 1
 
    for t in reversed(range(T-1)):
        for n in range(N):
            X[n,t] = np.sum(X[:,t+1] * self.A[n,:] * self.B[:, obs_seq[t+1]])
 
    return X
```

## 监督学习方法

假设已给训练数据包含S个长度相同的观测序列和对应的状态序列$\{(O_1,I_1),(O_2,I_2),\cdots,(O_S,I_S)\}$，那么可以利用极大似然估计法来估计隐马尔可夫模型的参数。具体方法如下。

1. 转移概率 $a_{ij}$ 的估计
设样本中时刻t处于状态i时刻t+1转移到状态j的频率为 $A_{ij}$，那么状态转移概率 $a_{ij}$ 的估计是 $$\hat{a_{ij}} = \frac{A_{ij}}{\sum^N_{j=1}A_{ij}}, i=1,2,\cdots,N$$ 很简单的最大似然估计。
2. 观测概率的估计
设样本中状态为j并观测为k的频数是 $B_{jk}$，那么状态为j观测为k的概率的估计是 $$\hat{b_j}(k) = \frac{B_{jk}}{\sum_{k=1}^MB_{jk}}, j=1,2,\cdots,N; k=1,2,\cdots,M$$
3. 初始状态概率 $\pi_i$ 的估计 $\hat{\pi_i}$ 为 S 个样本中初始化状态为 $q_i$ 的概率。

由于监督学习需要使用训练数据，而人工标注训练数据往往代价很高，有时就会利用非监督学习的方法。

### Baum-Welch算法

假设给定训练数据只包含S个长度为T的观测序列 $\{O_1,O_2,\cdots,O_S\}$ 而没有对应的状态序列，目标是学习隐马尔可夫模型 $\lambda=(A,B,\pi)$ 的参数。我们将观测序列数据看作观测数据O，状态序列数据看作不可观测的隐数据I，那么隐马尔可夫模型事实上是一个含有隐变量的概率模型 $$P(O|\lambda)=\sum_IP(O|I,\lambda)P(I|\lambda)$$ 它的参数学习可以由 EM 算法实现。

1. 确定完全数据的对数似然函数
2. EM算法的E步：求Q函数 $Q(\lambda,\overline{\lambda})$ $$Q(\lambda,\overline{\lambda})=\sum_I \log P(O,I|\lambda)P(O,I|\overline{\lambda})$$ 其中，$\overline{\lambda}$ 是隐马尔可夫模型参数的当前估计值，$\lambda$ 是要极大化的隐马尔可夫模型参数。（Q函数的标准定义是：$Q(\lambda,\overline{\lambda})=E_I[\log P(O,I|\lambda)|O,\overline{\lambda}]$，式子内部其实是条件概率，其中的 $O, \overline{\lambda}$ 对应 $\frac{1}{P(O|\overline{\lambda})}$；其与 $\lambda$ 无关，所以省略掉了。） $$P(O,I|\lambda)=\pi_{i_1}b_{i_1}(o_1)a_{i_1i_2}b_{i_2}(o_2)\cdots a_{i_{T-1}i_T}b_{i_T}(o_T)$$
这个式子从左到右依次是初始概率、发射概率、转移概率、发射概率……
于是函数$Q(\lambda,\overline{\lambda})$可以写成：
$$
Q(\lambda,\overline{\lambda})=\sum_I \log \pi_{i_1}P(O,I|\overline{\lambda}) \\
+ \sum_I \left(\sum_{t=1}^{T-1} \log a_{i_t i_{t+1}} \right) P(O,I|\overline{\lambda}) \\
+ \sum_I \left(\sum_{t=1}^T \log b_{i_t}(o_t) \right) P(O,I|\overline{\lambda})
$$
式中求和都是对所有训练数据的序列总长度T进行的。这个式子是将 $P(O,I|\lambda)=\pi_{i_1}b_{i_1}(o_1)a_{i_1i_2}b_{i_2}(o_2)\cdots a_{i_{T-1}i_T}b_{i_T}(o_T)$ 代入 $Q(\lambda,\overline{\lambda})=\sum_I \log P(O,I|\lambda)P(O,I|\overline{\lambda})$ 后，将初始概率、转移概率、发射概率这三部分乘积的对数拆分为对数之和，所以有三项。
3. EM算法的M步:极大化Q函数$Q(\lambda,\overline{\lambda})$求模型参数$A,B,\pi$，由于要极大化的参数在Q函数表达式中单独地出现在3个项中，所以只需对各项分别极大化。
**第1项可以写成：**
$$\sum_I \log \pi_{i_0}P(O,I|\overline{\lambda})=\sum_{i=1}^N \log \pi_i P(O,i_1=i|\overline{\lambda})$$
注意到$\pi_i$满足约束条件利用拉格朗日乘子法，写出拉格朗日函数：
$$\sum_{i=1}^N \log \pi_i P(O,i_1=i|\overline{\lambda}) + \gamma \left(\sum_{i=1}^N \pi_i - 1 \right)$$
对其求偏导数并令结果为0
$$\frac{\partial}{\partial\pi_i} \left[\sum_{i=1}^N \log \pi_i P(O,i_1=i|\overline{\lambda}) + \gamma \left(\sum_{i=1}^N \pi_i - 1 \right) \right] = 0$$
得到
$$P(O,i_1=i|\overline{\lambda})+\gamma\pi_i=0$$
这个求导是很简单的，求和项中非i的项对πi求导都是0，logπ的导数是1/π，γ那边求导就剩下πi自己对自己求导，也就是γ。等式两边同时乘以πi就得到了上式。
对i求和得到γ：
$$\gamma=-P(O|\overline{\lambda})$$
代入 $P(O,i_1=i|\overline{\lambda})+\gamma\pi_i=0$ 中得到：
$$\pi_i=\frac{P(O,i_1=i|\overline{\lambda})}{P(O|\overline{\lambda})}$$
**第2项可以写成：**
$$\sum_I \left(\sum_{t=1}^{T-1} \log a{i_t i_{t+1}} \right) P(O,I|\overline{\lambda}) = \sum_{i=1}^N \sum_{j=1}^N \sum_{t=1}^{T-1} \log a_{ij} P(O,i_t=i,i_{t+1}=j|\overline{\lambda})$$
类似第1项，应用具有约束条件 $\sum_{j=1}^N a_{ij}=1$ 的拉格朗日乘子法可以求出
$$a_{ij}=\frac{\sum_{t=1}^{T-1}P(O,i_t=i,i_{t+1}=j|\overline{\lambda})}{\sum_{t=1}^{T-1}P(O,i_t=i|\overline{\lambda})}$$
**第3项为：**
$$\sum_I \left(\sum_{t=1}^T \log b_{i_t}(o_t) \right)P(O,I|\overline{\lambda}) = \sum_{j=1}^N \sum_{t=1}^T \log b_j(o_t)P(O,i_t=j|\overline{\lambda})$$

同样用拉格朗日乘子法，约束条件是$\sum_{k=1}^M b_j(k)=1$。注意，只有在对$O_t=V_k$时$b_j(o_t)$对$b_j(k)$的偏导数才不为0,以$I(o_t=v_k)$表示。求得
$$b_j(k)=\frac{\sum_{t=1}^T P(O,i_t=j|\overline{\lambda})I(o_t=v_k)}{\sum_{t=1}^TP(O,i_t=j|\overline{\lambda})}$$

#### Baum-Welch模型参数估计公式

将这三个式子中的各概率分别简写如下：

$$\gamma_t(i)=\frac{\alpha_t(i)\beta_t(i)}{P(O|\lambda)}=\frac{\alpha_t(i)\beta_t(i)}{\sum_{j=1}^N \alpha_t(j) \beta_t(j)}$$
$$\xi_t(i,j)=\frac{\alpha_t(i)a_{ij}b_j(o_{t+1})\beta_{t+1}(j)}{\sum_{i=1}^N \sum_{j=1}^N \alpha_t(i)a_{ij}b_j(o_{t+1})\beta_{t+1}(j)}$$

则可将相应的公式写成：

$$a_{ij}=\frac{\sum_{t=1}^{T-1} \xi_t(i,j)}{\sum_{t=1}^{T-1} \gamma_t(i)}$$
$$b_j(k)=\frac{\sum_{t=1,o_t=v_k}^T \gamma_t(j)}{\sum_{t=1}^T \gamma_t(j)}$$
$$\pi_i=\gamma_1(i)$$

这三个表达式就是Baum-Welch算法（Baum-Welch algorithm)，它是EM算法在隐马尔可夫模型学习中的具体实现，由Baum和Welch提出。

#### 算法推演

输入：观测数据 $O=(o_1,o_2,\cdots,o_T)$

输出：隐马尔可夫模型参数

1. 初始化
对 $n=0$，选取 $a_{ij}^{(0)},b_j(k)^{(0)},\pi_i^{(0)}$，得到模型 $\lambda^{(0)}=(A^{(0)},B^{(0)},\pi^{(0)})$。
2. 递推。对$n=1,2,\cdots,$ 
$$a_{ij}^{n+1}=\frac{\sum_{t=1}^{T-1} \xi_t(i,j)}{\sum_{t=1}^{T-1} \gamma_t(i)}$$
$$b_j(k)^{n+1}=\frac{\sum_{t=1,o_t=v_k}^T \gamma_t(j)}{\sum_{t=1}^T \gamma_t(j)}$$
$$\pi_i^{(n+1)}= \gamma_1(i)$$
右端各值按观测$O=(o_1,o_2,\cdots,o_T)$ 和模型 $\lambda^{(n)}=(A^{(n)},B^{(n)},\pi^{(n)})$ 计算。
3. 终止。得到模型参数 $\lambda^{(n+1)}=(A^{(n+1)},B^{(n+1)},\pi^{(n+1)})$

#### Baum-Welch算法的Python实现

```py
def baum_welch_train(self, observations, criterion=0.05):
    n_states = self.A.shape[0]
    n_samples = len(observations)

    done = False
    while not done:
        # alpha_t(i) = P(O_1 O_2 ... O_t, q_t = S_i | hmm)
        # Initialize alpha
        alpha = self._forward(observations)

        # beta_t(i) = P(O_t+1 O_t+2 ... O_T | q_t = S_i , hmm)
        # Initialize beta
        beta = self._backward(observations)

        xi = np.zeros((n_states,n_states,n_samples-1))
        for t in range(n_samples-1):
            denom = np.dot(np.dot(alpha[:,t].T, self.A) * self.B[:,observations[t+1]].T, beta[:,t+1])
            for i in range(n_states):
                numer = alpha[i,t] * self.A[i,:] * self.B[:,observations[t+1]].T * beta[:,t+1].T
                xi[i,:,t] = numer / denom

        # gamma_t(i) = P(q_t = S_i | O, hmm)
        gamma = np.sum(xi,axis=1)
        # Need final gamma element for new B
        prod =  (alpha[:,n_samples-1] * beta[:,n_samples-1]).reshape((-1,1))
        gamma = np.hstack((gamma,  prod / np.sum(prod))) #append one more to gamma!!!

        newpi = gamma[:,0]
        newA = np.sum(xi,2) / np.sum(gamma[:,:-1],axis=1).reshape((-1,1))
        newB = np.copy(self.B)

        num_levels = self.B.shape[1]
        sumgamma = np.sum(gamma,axis=1)
        for lev in range(num_levels):
            mask = observations == lev
            newB[:,lev] = np.sum(gamma[:,mask],axis=1) / sumgamma

        if np.max(abs(self.pi - newpi)) < criterion and \
                        np.max(abs(self.A - newA)) < criterion and \
                        np.max(abs(self.B - newB)) < criterion:
            done = 1

        self.A[:],self.B[:],self.pi[:] = newA,newB,newpi
```

代码有点长，一段一段地看。

先拿到前后向概率：

```
alpha = self._forward(observations)
beta = self._backward(observations)
```

然后计算 $$\xi_t(i,j)=\frac{\alpha_t(i)a_{ij}b_j(o_{t+1})\beta_{t+1}(j)}{\sum_{i=1}^N \sum_{j=1}^N \alpha_t(i)a_{ij}b_j(o_{t+1})\beta_{t+1}(j)}$$

```py
xi = np.zeros((n_states,n_states,n_samples-1))
for t in range(n_samples-1):
    denom = np.dot(np.dot(alpha[:,t].T, self.A) * self.B[:,observations[t+1]].T, beta[:,t+1])
    for i in range(n_states):
        numer = alpha[i,t] * self.A[i,:] * self.B[:,observations[t+1]].T * beta[:,t+1].T
        xi[i,:,t] = numer / denom
```

注意xi(ξ)的下标t少了一个，这是因为对于t=T，没法用t+1去定位后向概率。所以这一个时刻是这么计算的：

```py
# gamma_t(i) = P(q_t = S_i | O, hmm)
gamma = np.sum(xi,axis=1)
# Need final gamma element for new B
prod =  (alpha[:,n_samples-1] * beta[:,n_samples-1]).reshape((-1,1))
gamma = np.hstack((gamma,  prod / np.sum(prod))) #append one more to gamma!!!
```

gamma有了，于是$\pi_i^{(n+1)}= \gamma_1(i)$取下标1得到新的pi：

```py
newpi = gamma[:,0]
```

xi(ξ)求和除以gamma(γ)求和得到新的A：

$$a_{ij}^{n+1}=\frac{\sum_{t=1}^{T-1} \xi_t(i,j)}{\sum_{t=1}^{T-1} \gamma_t(i)}$$

```py
newA = np.sum(xi,2) / np.sum(gamma[:,:-1],axis=1).reshape((-1,1))
```

利用下式得到新的B：

$$b_j(k)^{n+1}=\frac{\sum_{t=1,o_t=v_k}^T \gamma_t(j)}{\sum_{t=1}^T \gamma_t(j)}$$

```py
num_levels = self.B.shape[1]
sumgamma = np.sum(gamma,axis=1)
for lev in range(num_levels):
    mask = observations == lev
    newB[:,lev] = np.sum(gamma[:,mask],axis=1) / sumgamma
```

接着检查是否满足终止阈值，否则继续下一轮训练。

回到诊所的例子，我们可以用这样一串代码完成Baum-Welch算法的训练，并且评估其准确率：

```py
# run a baum_welch_train
observations_data, states_data = h.simulate(100)
# print observations_data
# print states_data
guess = hmm.HMM(np.array([[0.5, 0.5],
                          [0.5, 0.5]]),
                np.array([[0.3, 0.3, 0.3],
                          [0.3, 0.3, 0.3]]),
                np.array([0.5, 0.5])
                )
guess.baum_welch_train(observations_data)
states_out = guess.state_path(observations_data)[1]
p = 0.0
for s in states_data:
    if next(states_out) == s: p += 1

print p / len(states_data)
```

输出：

```py
0.58
```

视simulate出来的随机数据不同，准确率在40%-70%之间波动。这其实说明对于这个例子，无监督学习并不靠谱，只能全靠创业团队的PPT了。

另外，由于这是一份来自[colostate大学](http://www.cs.colostate.edu/~anderson/cs440/index.html/doku.php?id=notes:hmm2)的教学代码，全部运算都是浮点数乘法，没有取对数，所以在数据量较大的时候可能发生除零错误。

## 预测算法

下面介绍隐马尔可夫模型预测的两种算法：近似算法与维特比算法（Viterbi algorithm)。

### 近似算法

近似算法的想法是，在每个时刻t选择在该时刻最有可能出现的状态$i_t^*$，从而得到一个状态序列$I^*=(i_1^*,i_2^*,\cdots,i_T^*)$，将它作为预测的结果。

给定隐马尔可夫模型 $\lambda$ 和观测序列 $O$，在时刻t处于状态 $q_i$ 的概率 $\gamma_t(i)$ 是 
$$\gamma_t(i)=\frac{\alpha_t(i)\beta_t(i)}{P(O|\lambda)}=\frac{\alpha_t(i)\beta_t(i)}{\sum_{j=1}^N \alpha_t(j) \beta_t(j)}$$

在每一时刻t最优可能的状态$i_t^*$是 $$i_t^*=\arg \max_{1\leq i\leq N}[\gamma_t(i)], t=1,2,\cdots,T$$ 从而得到状态序列 $I^*=(i_1^*,i_2^*,\cdots,i_T^*)$。

近似算法的优点是计算简单，其缺点是不能保证预测的状态序列整体是最有可能的状态序列，因为预测的状态序列可能有实际不发生的部分。事实上，上述方法得到的状态序列中有可能存在转移概率为0的相邻状态，即对某些$a_{ij}=0$时。尽管如此，近似算法仍然是有用的（没看出来有什么用）。

### 维特比算法

维特比算法实际是用动态规划解隐马尔可夫模型预测问题，即用动态规划(dynamic programming)求概率最大路径（最优路径）。这时一条路径对应着一个状态序列。


文章来自：

> http://www.hankcs.com/ml/hidden-markov-model.html

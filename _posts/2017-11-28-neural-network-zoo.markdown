---
layout: post
title: 神经网络结构表
subtitle: 来自 Asimov 人工智能研究所的 neural network zoo
category: Document
tags: neural
date: 2017-11-28 15:11:09
summary: 受打击了，巨人的肩膀不好爬啊！
cover: 'http://www.asimovinstitute.org/wp-content/uploads/2016/09/neuralnetworks.png'
mathjax: false
highlight: false
---

> 面试官：请画一下 GRU 的网络图，并讲一下它的工作原理。

> 应聘者：（尼玛，你大爷知道什么是 GRU）...呃...GRU 是？

> 面试官：好走，不送！

> 应聘者：（别TM给我提什么狗屁原理，老子写代码就是一把梭，复制加粘贴）...

新的神经网络结构不断涌现，我们很难一一掌握。哪怕一开始只是记住所有的简称（ DCIGN，BiLSTM，DCGAN ），也会让同学们吃不消。

所以我决定写篇文章归纳归纳，各种神经网络结构。它们大部分都是神经网络，也有一些是完全不同的结构。虽然所有结构说起来都是新颖而独特的，但当我画出结点的结构图时……它们之间的内在联系显得更有意思。

![](http://www.asimovinstitute.org/wp-content/uploads/2016/09/neuralnetworks.png)

*总表*

当我们在画节点图的时候发现了一个问题：这些图并没有展示出来这些神经结构是怎么使用的。

打个比方，变分自编码器（ VAE ）看起来跟自编码器（ AE ）真的差不多，但它们的训练过程却相差很远。在使用训练好的网络时更为不同，因为 VAE 是生成器，是通过添加噪音来获得新样本的。而 AE 只是简单地通过搜索记忆，找到与输入最接近的训练样本。

我需要强调的是，这个图并不能反映不同节点结构内部是如何运作的（这已经不是今天的话题了）。

需要注意的是，虽然我们使用的大部分简称是被普遍接受的，仍有一些并非如此。RNN 有的时候指的是递归神经网络（ recursive neural networks ），但大部分时候指的是循环神经网络（ recurrent neural networks ）。有时能看到 RNN 会被用来代表任何循环结构，包括 LSTM， GRU 甚至是双向网络的变型。 AE 偶尔也有类似的问题， VAE 和DAE 有时也被称为 AE 。许多简称在末尾的 N 的数量上有区别，因为你可以管卷积神经网络（ convolutional neural network ）叫成卷积网络（ convolutional network ）（就会形成两种简称： CNN 和 CN ）。

因为新的网络一直在不断地出现，构建一张完整的列表实际上是不可能的。就算是有这么一张表格，你在表格里找某一个也相当吃力，有时你会忽视掉一部分。所以虽然这张列表或许可以为你打开人工智能世界的大门， 但决不能认为这张表格列出了所有的结构，特别是当这篇文章写完很久之后，你才读到它。

对每个图片里的结构，我都写了一个非常非常简短的说明。如果你对某类结构很熟悉但对具体某个结构不熟的话，你可能会觉得这些说明挺有用的。

--------

## 1.前馈神经网络（FF或FFNN）和感知机（P）

![前馈神经网络（FF或FFNN）和感知机（P）](http://www.asimovinstitute.org/wp-content/uploads/2016/09/ff.png)


**前馈神经网络（ FF 或 FFNN ）和感知机（ P ）**十分直接，信息从前端一路传递到后端（从输入到输出）。神经网络通常有层的概念，每层可能是输入单元，隐藏单元或者输出单元组成。单个层内没有连接，通常是两个相邻的层之间全连接，（一层的所有神经元都和另一层的所有神经元互相连接）。

最简单但有效的神经网络有两个输入单元，一个输出单元，可以用于构建逻辑门。通常给网络的是成对的信息，输入和我们希望的输出，用反向传播来训练 FFNN 。这种叫做监督学习，和无监督学习相反，无监督学习是我们只给网络输入，让网络自己填补空缺。反向传播的误差通常是模型给定的输出和应该的输出之间的差距的某种变型（比如说 MSE 或者就是线性的做差）。如果网络有足够多的隐藏神经元，理论上它总可以给出输入和输出之间的关系建模。实际运用它们中有一定的局限性，但经常和其他网络组合起来构成新的网络。

*Rosenblatt, Frank. “The perceptron: a probabilistic model for information storage and organization in the brain.” Psychological review 65.6 (1958): 386.*

[原始论文](http://www.ling.upenn.edu/courses/cogs501/Rosenblatt1958.pdf)

---------

## 2.径向基函数（·RBF·）网络

![Radial basis function(RBF)](http://www.asimovinstitute.org/wp-content/uploads/2016/09/rbf.png)


**径向基函数（ RBF ）网络**是 FFNN 网络附上径向基函数作为激活函数。就只是这样而已。我不是说它没什么用，而是大部分用其他激活函数的 FFNN 没有专用的名字。 这个有名字主要只是因为它恰巧在对的时间被发现了。

*Broomhead, David S., and David Lowe. Radial basis functions, multi-variable functional interpolation and adaptive networks. No. RSRE-MEMO-4148. ROYAL SIGNALS AND RADAR ESTABLISHMENT MALVERN (UNITED KINGDOM), 1988.*

[原始论文](http://www.dtic.mil/cgi-bin/GetTRDoc?AD=ADA196234)

----------

## 3.Hopfield 网络（ HN ）


![Hopfield network (HN)](http://www.asimovinstitute.org/wp-content/uploads/2016/09/hn.png)


**Hopfield 网络（ HN ）**是一个每个神经元都和其他的神经元互相连接的网络； 它就是一盘彻底纠缠在一起的意大利面，每个节点也是如此。每个节点既是训练前的输入，又是训练中的隐藏节点，还是最后的输出。网络中的训练过程先计算出权重，然后每个神经元设置成我们想要的值。这个值将不再变动。一旦训练了一种或多种模式，网络最终会收敛到某个学习好的模式，因为网络只在这些状态上稳定。

请注意，网络不一定符合想要的结果。（可惜它并不是一个有魔法的黑箱子）它能够稳定部分源于网络的总能量或是随着训练逐渐降低的温度。每个神经元有一个对应于温度的激活阈值，如果输入的值加起来的和超过了这个阈值，神经元就会返回两个状态之一（通常是 -1 或 1 ，有时是 0 或 1 ）。网络的更新可以同步完成，更通常的做法是一个一个更新参数。如果一个一个更新的话，会产生一个公平随机的序列来决定单元更新的顺序（公平随机指的是一共 n 个选项一个个地更新，一共循环 n 次）。

所以你可以说当每个单元都更新过而且每个都不变了，这个网络稳定（退火）了（完成了收敛）。这个网络通常被称为联想记忆因为它能够收敛到和输入最相近的状态。如果人们能看到一半的桌子，我们能想出另一半桌子，同样地这个网络也会由残缺的半个桌子和一些噪音收敛到一个完整的桌子。

*Hopfield, John J. “Neural networks and physical systems with emergent collective computational abilities.” Proceedings of the national academy of sciences 79.8 (1982): 2554-2558.*

[原始论文](https://bi.snu.ac.kr/Courses/g-ai09-2/hopfield82.pdf)

--------

## 4.马尔可夫链（·MC·或离散时间马尔可夫链，·DTMC·）

![Markov chains (MC or discrete time Markov Chain, DTMC)](http://www.asimovinstitute.org/wp-content/uploads/2016/09/mc.png)

**马尔可夫链（ MC 或离散时间马尔可夫链， DTMC ）**是 BM 和 HN 的前身。它可以这样理解：从我在的节点，到达我周围的节点的概率是多少？它们是无记忆的（这就是马尔可夫性质），这意味着下一状态转移到哪里完全依赖于上一个状态。这不能算是一个神经网络，但是有一点类似神经网络，而且是 BM 和 HN 的理论基础。 MC 通常我们认为它不是一个神经网络， BM，RBM，HN 也是。马尔可夫链不一定是全连接的。

*Hayes, Brian. “First links in the Markov chain.” American Scientist 101.2 (2013): 252.*

[原始论文](http://www.americanscientist.org/libraries/documents/201321152149545-2013-03Hayes.pdf)

------

## 5.玻尔兹曼机（ BM ）

![Boltzmann machines (BM)](http://www.asimovinstitute.org/wp-content/uploads/2016/09/bm.png)


**玻尔兹曼机（ BM ）**和 HN 非常相似，但有些神经元被标注为输入神经元，其他的仍然是隐藏的。这些输入神经元在完成了整个网络更新后会成为输出神经元。它从随机权重开始通过反向传播学习，最近也会使用对比散度（一个用来决定两个信息增益之间梯度的马尔科夫链）学习。对比 HN， 神经元通常有二元激活模式。由于使用 MC 训练，BM 是一个随机网络。 

BM 的训练和使用和 HN 很相似：把输入神经元设到箝位值然后这个网络就自由了（不用外力了）。这些单元在自由之后可以取任何值，我们在输入神经元和隐藏神经元间而反复迭代。激活模式是由全局温度控制的，更低的全局温度会降低单元的能量并令激活模式稳定下来。网络会在合适的温度下达到平衡点。

*Hinton, Geoffrey E., and Terrence J. Sejnowski. “Learning and releaming in Boltzmann machines.” Parallel distributed processing: Explorations in the microstructure of cognition 1 (1986): 282-317.*

[原始论文](https://www.researchgate.net/profile/Terrence_Sejnowski/publication/242509302_Learning_and_relearning_in_Boltzmann_machines/links/54a4b00f0cf256bf8bb327cc.pdf)

------

## 6.受限玻尔兹曼机（ RBM ）

![Restricted Boltzmann machines (RBM)](http://www.asimovinstitute.org/wp-content/uploads/2016/09/rbm.png)


**受限玻尔兹曼机（ RBM ）**和 BM 非常类似（惊不惊喜）所以也和 HN 类似。RBM 和 BM最大的区别是 RBM 更利于使用因为它是受限制的。它们不是每个神经元都和其他相连，而是每个组的神经元和别的组的神经元连接，所以没有输入神经元直接连接到其他输入神经元，也没有隐藏神经元连接到其他隐藏神经元。 RBM 可以类似于 FFNN 的训练方式来训练，但要做一点点改变：不是把信息从头传到末再反向传播误差，而是先把信息从头传到末，再把信息反向传回（到第一层）。然后接着开始用前向和反向传播来训练。

*Smolensky, Paul. Information processing in dynamical systems: Foundations of harmony theory. No. CU-CS-321-86. COLORADO UNIV AT BOULDER DEPT OF COMPUTER SCIENCE, 1986.*

[原始论文](http://www.dtic.mil/cgi-bin/GetTRDoc?Location=U2&doc=GetTRDoc.pdf&AD=ADA620727)



# 参考：

> http://www.asimovinstitute.org/neural-network-zoo/
> https://mp.weixin.qq.com/s/FlS6D5jaPJ_Ai_7G-87S1g

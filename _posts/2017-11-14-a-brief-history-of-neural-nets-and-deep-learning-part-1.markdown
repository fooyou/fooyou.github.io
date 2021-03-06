---
layout: post
title: 神经网络和深度学习简史（一）
subtitle: 50年代神经网络的诞生和70年代的寒冬及80年代的复苏
category: Document
tags: ml neural
date: 2017-11-15 09:11:47
summary: 如今，深度学习浪潮拍打计算机语言的海岸已有好几年，但是，2015 年似乎才是这场海啸全力冲击自然语言处理（NLP）会议的一年。—— Dr. Christopher D. Manning, Dec 2015
cover: 'https://draftin.com/images/34998?token=eP9x7J-SbQvC30KjZ23yt38XOWoU6_d0JVo72rZF-EHtWcLW-zNPyZU2ZYJu4VPm7Fxs20Gd7mPoRylRtNFigXs'
mathjax: false
highlight: false
---

本部分介绍神经网络和感知机在1958年的诞生，和70年代的寒冬和1986年反向传播让它重新受欢迎的历史。

**关于这篇博文**

从事神经网络和深度学习也有一两年的时间了，但一直没有从历史发展脉络上把握这波技术浪潮，偶然发现了这篇有斯坦福大学的 26 岁学生 Andrey Kurenkov 写的博文，看后颇有收获，特此重新整理一下，以供加深印象。好了，正文开始：

# 第一部分：神经网络的诞生和寒冬

## 深度学习掀起海啸

> 如今，深度学习浪潮拍打计算机语言的海岸已有好几年，但是，2015年似乎才是这场海啸全力冲击自然语言处理（NLP）会议的一年。——[Dr. Christopher D. Manning, Dec 2015](http://www.mitpressjournals.org/doi/pdf/10.1162/COLI_a_00239)[^1]

整个研究领域的成熟方法已经迅速被新发现超越，这句话听起来有些夸大其词，就像是说它被「海啸」袭击了一样。但是，这种灾难性的形容的确可以用来描述深度学习在过去几年中的异军突起——显著改善人们对解决人工智能最难问题方法的驾驭能力，吸引工业巨人（比如谷歌等）的大量投资，研究论文的指数式增长（以及机器学习的研究生生源上升）。在听了数节机器学习课堂，甚至在本科研究中使用它以后，我不禁好奇：这个新的「深度学习」会不会是一个幻想，抑或上世纪80年代已经研发出来的「人工智能神经网络」扩大版？让我告诉你，说来话长——这不仅仅是一个有关神经网络的故事，也不仅仅是一个有关一系列研究突破的故事，这些突破让深度学习变得比「大型神经网络」更加有趣，而是一个有关几位不放弃的研究员如何熬过黑暗数十年，直至拯救神经网络，实现深度学习梦想的故事。

**声明：我不是专家，更多信息及更正**

我还没有能力成为这个话题的专家。在深入的技术概率中，请参考这些实际创造了这个领域的大牛们所写的参考书目：

- [Learning Deep Architectures for AI](http://www.iro.umontreal.ca/~lisa/pointeurs/TR1312.pdf) Yoshua Bengio
- [Deep Learning](http://www.cs.toronto.edu/~hinton/absps/NatureDeepReview.pdf) LeCun et al

    *这个几乎是北美的 AI 社区的研究历史，即便如此，还是有很多的研究者没有被提及。*

- [Deep learning in Neural Networks: An Overview](http://arxiv.org/pdf/1404.7828v4.pdf) Jürgen Schmidhuber

    这个关于 AI 的深度的历史补足了 LeCun Deep Learning 里未涉及的领域


## 机器学习算法的百年历史

![线性回归](https://upload.wikimedia.org/wikipedia/commons/3/3a/Linear_regression.svg)


首先简单介绍一下机器学习是什么。从二维图像上取一些点，尽可能绘出一条拟合这些点的直线。你刚才做的就是从几对输入值（x）和输出值（y）的实例中概括出一个通用函数，任何输入值都会有一个对应的输出值。这叫做线性回归，一个有着[200年历史](https://en.wikipedia.org/wiki/Linear_regression#cite_note-4)从一些输入输出对组中推断出一般函数的技巧。这就是它很棒的原因：很多函数难以给出明确的方程表达，但是，却很容易在现实世界搜集到输入和输出值实例——比如，将说出来的词的音频作为输入，词本身作为输出的映射函数。

**线性回归**对于解决语音识别这个问题来说有点太无用，但是，它所做的基本上就是**监督式机器学习**：给定**训练样本**，「学习」一个函数，每一个样本数据就是需要学习的函数的输入输出数据（无监督学习，稍后在再叙）。尤其是，机器学习应该推导出一个函数，它能够很好地泛化到不在训练集中的输入值上，既然我们真的能将它运用到尚未有输出的输入中。例如，谷歌的语音识别技术由拥有大量训练集的机器学习驱动，但是，它的训练集也不可能大到包含你手机所有语音输入。

泛化能力机制如此重要，以至于总会有一套**测试数据组**（更多的输入值与输出值样本）这套数据组并不包括在训练组当中。通过观察有多少个正确计算出输入值所对应的输出值的样本，这套单独数据组可以用来估测机器学习技术有效性。概括化的克星是**过度拟合**——学习一个对于训练集有效但是却在测试数据组中表现很差的函数。既然机器学习研究者们需要用来比较方法有效性的手段，随着时间的推移，标准训练数据组以及测试组可被用来评估机器学习算法。

好了，定义谈得足够多了。重点是——我们绘制线条的联系只是一个非常简单的监督机器学习例子：要点在于训练集（X为输入，Y为输出），线条是近似函数，用这条线来为任何没有包含在训练集数据里的X值（输入值）找到相应的Y值（输出值）。别担心，接下来的历史就不会这么干巴巴了。让我们继续吧。


### 虚假承诺的荒唐

显然这里话题是神经网络，那我们前言里为何要扯线性回归呢？呃, 事实上线性回归和机器学习一开始的方法构想：[弗兰克 罗森布拉特(Frank Rosenblatt)的**感知机**](http://psycnet.apa.org/index.cfm?fa=buy.optionToBuy&id=1959-09865-001)[^2], 有些许相似性。

![感知机工作图](https://draftin.com/images/34998?token=eP9x7J-SbQvC30KjZ23yt38XOWoU6_d0JVo72rZF-EHtWcLW-zNPyZU2ZYJu4VPm7Fxs20Gd7mPoRylRtNFigXs)

心理学家Rosenblatt构想了感知机，它作为简化的数学模型解释大脑神经元如何工作：它取一组二进制输入值（附近的神经元），将每个输入值乘以一个连续值权重（每个附近神经元的突触强度），并设立一个阈值，如果这些加权输入值的和超过这个阈值，就输出1，否则输出0（同理于神经元是否放电）。对于感知机，绝大多数输入值不是一些数据，就是别的感知机的输出值。但有一个额外的细节：这些感知机有一个特殊的，输入值为1的，「偏置」输入，因为我们能补偿加权和，它基本上确保了更多的函数在同样的输入值下是可计算的。这一关于神经元的模型是建立在沃伦·麦卡洛克(Warren McCulloch)和沃尔特·皮兹(Walter Pitts)工作上的（[Mcculoch-Pitts](http://www.minicomplexity.org/pubs/1943-mcculloch-pitts-bmb.pdf)）[^3]。他们曾表明，把二进制输入值加起来，并在和大于一个阈值时输出1，否则输出0的神经元模型，可以模拟基本的或/与/非逻辑函数。这在人工智能的早期时代可不得了——当时的主流思想是,计算机能够做正式的逻辑推理将本质上解决人工智能问题。

![感知机工作图 2](https://draftin.com/images/34832?token=vQiHNdPnUSPiPJcJcgobMGedDJRvgguccVapCN76gZnxqVQIKczfq4BqUQ06bWdVXnabb3tScv_04nigKqMZjS4)

*另一个图表，显示出生物学上的灵感。激活函数就是人们当前说的非线性函数，它作用于输入值的加权和以产生人工神经元的输出值——在罗森布拉特的感知机情况下，这个函数就是输出一个阈值操作*


然而，麦卡洛克-皮兹模型缺乏一个对AI而言至关重要的学习机制。这就是感知机更出色的地方所在——罗森布拉特受到唐纳德·赫布(Donald Hebb) [基础性工作](http://onlinelibrary.wiley.com/doi/10.1002/cne.900930310/abstract)[^4]的启发，想出一个让这种人工神经元学习的办法。赫布提出了一个出人意料并影响深远的想法，称知识和学习发生在大脑主要是通过神经元间突触的形成与变化，简要表述为赫布法则：

> “当细胞A的轴突足以接近以激发细胞B，并反复持续地对细胞B放电，一些生长过程或代谢变化将发生在某一个或这两个细胞内，以致A作为对B放电的细胞中的一个，效率增加。”

感知机并没有完全遵循这个想法，但通过调输入值的权重，可以有一个非常简单直观的学习方案：给定一个有输入输出实例的**训练集**，感知机应该「学习」一个函数：对每个例子，若感知机的输出值比实例低太多，则增加它的权重，否则若设比实例高太多，则减少它的权重。更正式一点儿的该算法如下:

1. 从一个有着随机权重和训练集的感知机开始；
2. 对训练集中的一个实例输入，计算感知机的输出；
3. 如果感知机的输出值和训练集实例的默认结果不同，则：
    - 如果输出应为 0 但却为 1，减少输出为 1 的权重；
    - 如果输出应该 1 但却为 0，增减输出为 1 的权重；
4. 跳到训练集中下一个实例并重复 2-4 直到感知机不再犯错

这个过程很简单，产生了一个简单的结果：一个输入线性函数（加权和），就像线性回归一样，被非线性**激活函数**（总和的阈值）「压扁」了。当函数的输出值是一个有限集时（例如逻辑函数，它只有两个输出值True/1 和 False/0），给带权重的和设置阈值是没问题的，所以问题实际上不在于要对任何输入数据集生成一个数值上连续的输出（即回归类问题），而在于对输入数据做好合适的标签—**分类问题**。

![康奈尔航天实验室的Mark I 感知机](https://upload.wikimedia.org/wikipedia/en/5/52/Mark_I_perceptron.jpeg)

*第一台感知机硬件*

罗森布拉特用定制硬件的方法实现了感知机的想法（在花哨的编程语言被广泛使用之前），展示出它可以用来学习对20×20像素输入中的简单形状进行正确分类。自此，机器学习问世了——建造了一台可以从已知的输入输出对中得出近似函数的计算机。在这个例子中，它只学习了一个小玩具般的函数，但是从中不难想象出有用的应用，例如将人类乱糟糟的手写字转换为机器可读的文本。

但，等下！到这为止，我们只看到一个感知机如何学习输出一个1或0，这如何扩展至多分类任务呢？比如手写字（它有好多像分类类目的字符和数字）这对一台感知机来说是不可能完成的，因为它只有一个输出，但是，多输出函数能用位于同一**层（layer）**的多个感知机来学习，每个感知机接收到同一个输入，但分别负责函数的不同输出。实际上，神经网络（准确的说应该是「人工神经网络（ANN，Artificial Neural Networks）」）就是多层感知机（今天感知机通常被称为神经元）而已，只不过在这个阶段，只有一层——**输出层**。所以，神经网络的典型应用例子就是分辨手写数字。输入是图像的像素，有10个输出神经元，每一个分别对应着10个可能的数字。在这个案例中，10个神经元中，只有1个输出1，权值最高的和被看做是正确的输出，而其他的则输出0。

![多层输出神经网络](https://draftin.com/images/34466?token=YFsmpDuQfD3DDylinRD8F4sLOgjCFm4Aow1gIWoCY5KED3bnQKs17RaTja95OIQQWdr25dqS2fxq_6mDwwdcs9Y)

*多层输出神经网络*

也可以想象一个与感知机不同的人工神经网络。例如，阈值激活函数并不是必要的； 1960年，Bernard Widrow和Tedd Hoff很快开始探索一种方法——《[An adaptive “ADALINE” neuron using chemical “memistors”](http://www-isl.stanford.edu/~widrow/papers/t1960anadaptive.pdf)》[^5]，并展示了这种「自适应线性神经元」能够在电路中成为「 存储电阻器」的一部分（存储电阻器是带有存储的电阻）。他们还展示了，不用阈值激活函数，在数学上很美，因为神经元的学习机制可以正式地建立在通过良好的微积分来最小化误差的基础上。可以看出，通过从 0 到 1 的这个锐化的阈值跳跃，神经元的功能不会变的怪异，衡量每个权重变化（导数）时误差变化的程度可以用来驱动误差，找到最佳的权重值。正如我们看到的，使用每个权重的训练误差的导数来找到正确的权重正是现在神经网络的经典训练方式。

*附：更多点的数学知识*

> 简言之，如果一个函数是平滑的，则函数是可微的——Rosenblatt 的感知机的计算输出方式是：如果输入超过某个数字，输出会突然从 0 跳到 1，而 Adaline 只是输出非常好的非跳线输入。要更深入了解这些数学知识。可以读这篇[教程](http://sebastianraschka.com/Articles/2015_singlelayer_neurons.html)，或者 Google 一下。

如果我们多思考一下 「自适应（ADALINE）」，就会有进一步的洞见：为大量输入找到一组权重真的只是一种线性回归。再一次，就像用线性回归一样，这也不足以解决诸如语音识别或计算机视觉这样的人工智能难题。McCullough，Pitts和罗森布拉特真正感到兴奋的是联结主义（Connectionism）这个宽泛的想法：如此简单计算机单元构成的网络，其功能会大很多而且可以解决人工智能难题。而且罗森布拉特说的和（坦白说很可笑的）《纽约时报》[这段引文](http://www.nytimes.com/1958/07/08/archives/new-navy-device-learns-by-doing-psychologist-shows-embryo-of.html)的意思差不多：

> 海军披露了一台尚处初期的电子计算机，期待这台电子计算机能行走，谈话，看和写，自己复制出自身存在意识…罗森布拉特博士，康奈尔航空实验室的一位心理学家说，感知机能作为机械太空探险者被发射到行星上。

这种谈话无疑会惹恼人工领域的其他研究人员，其中有许多研究人员都在专注于这样的研究方法，它们以带有具体规则（这些规则遵循逻辑数学法则）的符号操作为基础。MIT人工智能实验室创始人Marvin Minsky和Seymour Paper就是对这一炒作持怀疑态度研究人员中的两位，1969年，他们在一本开创性著作中表达了这种质疑，书中严谨分析了感知机的局限性，书名很贴切，叫[感知机](https://mitpress.mit.edu/books/perceptrons`)[^7]。有趣的是， Minsky 实际上可能是第一个使用1951年的SNARC（随机神经模拟增强计算器）[^8]实现硬件神经网络的研究人员，这比 Rosenblatt 的工作要早好多年。但是由于他的工作缺乏跟进而且感知机的关键性质分析结果让他认为这种实现 AI 的方法是一条死胡同。他们分析中，最被广为讨论的内容就是对感知机限制的说明，例如，他们不能学习简单的布尔函数XOR，因为它不能进行**线性分离**。虽然此处历史模糊，但是，人们普遍认为这本书对人工智能步入第一个冬天起到了推波助澜的作用——大肆炒作之后，人工智能进入泡沫幻灭期，相关资助和出版都遭冻结。

![感知机局限性的视觉化](https://draftin.com/images/35002?token=NfMQ9LV1TFNi0v0QmbwfNTxYgbYjG7kjHqR9DaATySwhqdIE8Xc3Lfrcwa6JA2CAwSvQFYQL7fj_fwFT6o_Zelw)

*感知机局限性的视觉化。找到一个线性函数，输入X，Y时可以正确地输出+或-，就是在2D图表上画一条从+中分离出-的线；很显然，就第三幅图显示的情况来看，这是不可能的*

### 人工智能冬天的复苏

因此，情况对神经网络不利。但是，为什么？他们的想法毕竟是想将一连串简单的数学神经元结合在一起，完成一些复杂任务，而不是使用单个神经元。换句话说，并不是只有一个**输出层**，将一个输入任意传输到多个神经元（所谓的**隐藏层**，因为他们的输出会作为另一隐藏层或神经元输出层的输入）。只有输出层的输出是「可见」的——亦即神经网络的答案——但是，所有依靠隐藏层完成的间接计算可以处理复杂得多的问题，这是单层结构望尘莫及的。 

![有两个隐藏层的神经网络](https://draftin.com/images/34833?token=w5fXrfJQKt_CECpofd0YJUaAvNZEe2Qc0hgwlCj-x-48i8Bs5CtQDe52yns49AzIgK3NnSCwJvcmqQ1kp5mVwB0)

*有两个隐藏层的神经网络*

言简意赅地说，多个隐藏层是件好事，原因在于隐藏层可以找到数据内在**特征**，后续层可以在这些特点（而不是嘈杂庞大的原始数据）基础上进行操作。以图片中的面部识别这一非常常见的神经网络任务为例，第一个隐藏层可以获得图片的原始像素值，以及线、圆和椭圆等信息。接下来的层可以获得这些线、圆和椭圆等的位置信息，并且通过这些来定位人脸的位置——处理起来简单多了！而且人们基本上也都明白这一点。事实上，直到最近，机器学习技术都没有普遍直接用于原始数据输入，比如图像和音频。相反，机器学习被用于经过**特征提取**后的数据——也就是说，为了让学习更简单，机器学习被用在预处理的数据上，一些更加有用的特征，比如角度，形状早已被从中提取出来。

*附：为什么有非线性激活函数*

> 先前，我们看到由感知机计算的加权通常是一个非线性激活函数。现在我们回过头来解答一个隐含的问题：为什么那么麻烦？两个原因：1. 没有非线性激活函数，学习函数只能是线性的，而许多“有趣”的函数是非线性的（举例：只输出 0 和 1 的逻辑函数或者只输出类目的分类函数）；2. 由于所有计算的线性，几层线性感知机可能折叠成一层，而非线性激活函数则不会这样。
> 所以，直观上讲网络使用（非线性）激活函数可以更好的接管数据。


![传统手工提取特征过程](https://draftin.com/images/35001?token=qG2LYkSR81r2DESqTceIWvJdDBoXVK-PRShf3s-57FZbvr81A9xC9-4x46XNiD33WgEy9Kh3A95i09dYQnHdDrE)

因此，注意到这一点很重要：Minsky和Paper关于感知机的分析不仅仅表明不可能用单个感知机来计算XOR，而且特别指出需要多层感知机——亦即现在所谓的多层神经网络——才可以完成这一任务，而且罗森布拉特的学习算法对多层并不管用。那是一个真正的问题：之前针对感知机概括出的简单学习规则并不是适用于多层结构。想知道原因？让我们再来回顾一下单层结构感知机如何学习计算一些函数：

1. 以小的权重初始化和函数输出数相等的感知机开始；
2. 以训练集中的某个实例做为输入，计算感知机的输出；
3. 对每个感知机，如果其输出和训练集实例的结果不同，则调整其权值；
4. 对训练集中的其他实例，重复 2-4 过程，直到感知机不再犯错。

这一规则并不适用多层结构的原因应该很直观清楚了：选取训练集中的例子进行训练时，我们只能对最终的输出层的输出结果进行校正，但是，对于多层结构来说，我们该如何调整最终输出层之前的层结构权值呢？答案（尽管需要花时间来推导）又一次需要依赖古老的微积分：**链式法则**。这里有一个重要现实：**神经网络的神经元和感知机并不完全相同，但是，可用一个激活函数来计算输出，该函数仍然是非线性的，但是可微分，和Adaline神经元一样；该导数不仅可以用于调整权值，减少误差，链式法则也可用于计算前一层所有神经元导数，因此，调整它们权重的方式也是可知的。**说得更简单些：我们可以利用微积分将一些导致输出层任何训练集误差的原因分配给前一隐藏层的每个神经元，如果还有另外一层隐藏层，我们可以将这些原因再做分配，以此类推——我们在**反向传播**这些误差。而且，如果修改了神经网络（包括那些隐藏层）任一权重值，我们还可以找出误差会有多大变化，通过优化技巧（很长一段时间所使用的典型的i**随机梯度下降**）找出最小化误差的最佳权值。

![反向传播的基本思想](https://draftin.com/images/34948?token=LF6pwbG4bKjYLDLj4GDemUWKiFy8SQC5tluQgSGnxKxiaoFUlJ9FaYoC_Syh6t4fvzOT8rwz1fnBQ0xInJ_tuO0)


#### 反向传播

反向传播（Backpropagation）在上世纪 60 年代由多位研究人员提出，70 年代，由 Seppo Linnainmaa[^9] 引入电脑软件，Paul Werbos在1974年的博士毕业论文[^10]中深刻分析了将之用于神经网络方面的可能性，成为美国第一位提出可以将其用于神经网络的研究人员。有趣的是，他从模拟人类思维的研究工作中并没有获得多少启发，在这个案例中，弗洛伊德心理学理论启发了他，正如他[自己叙述](http://www.die.uchile.cl/ieee-cis/evic2005/files/AD2004Werbosv2.pdf)[^11]：

> 1968年，我提出我们可以多少模仿弗洛伊德的概念——信度指派的反向流动（ a backwards flow of credit assignment,），指代从神经元到神经元的反向流动…我解释过结合使用了直觉、实例和普通链式法则的反向计算，虽然它正是将弗洛伊德以前在心理动力学理论中提出的概念运用到数学领域中！

尽管解决了如何训练多层神经网络的问题，在写作自己的博士学位论文时也意识到了这一点，但是，Werbos没有发表将BP算法用于神经网络这方面的研究，直到1982年人工智能冬天引发了寒蝉效应。实际上，Werbos认为，这种研究进路对解决感知机问题是有意义的，但是，这个圈子大体已经失去解决那些问题的信念。

> Minsky的书最著名的观点有几个：（1）我们需要用MLPs(多层感知机，多层神经网络的另一种说法）来代表简单的非线性函数，比如XOR 映射；而且（2）世界上没人发现可以将MLPs训练得够好，以至于可以学会这么简单的函数的方法。Minsky的书让世上绝大多数人相信，神经网络是最糟糕的异端，死路一条。Widrow已经强调，这种压垮早期『感知机』人工智能学派的悲观主义不应怪在Minsky的头上。他只是总结了几百位谨慎研究人员的经验而已，他们尝试找出训练MLPs的办法，却徒劳无功。也曾有过希望，比如Rosenblatt所谓的backpropagation（这和我们现在说的 backpropagation并不完全相同！），而且Amari也简短表示，我们应该考虑将最小二乘（也是简单线性回归的基础）作为训练神经网络的一种方式（但没有讨论如何求导，还警告说他对这个方法不抱太大期望）。但是，当时的悲观主义开始变得致命。上世纪七十年代早期，我确实在MIT采访过Minsky。我建议我们合著一篇文章，证明MLPs实际上能够克服早期出现的问题…但是，Minsky并无兴趣（14）。事实上，当时的MIT，哈佛以及任何我能找到的研究机构，没人对此有兴趣。

似乎是因为缺乏学术兴趣，直到十多年后的 1986 年，这种方法才在 David Rumelhart、 Geoffrey Hinton 和 Ronald Williams[^12] 合著的《[Learning representations by back-propagating errors](http://www.iro.umontreal.ca/~vincentp/ift3395/lectures/backprop_old.pdf)》中流行开来。尽管研究方法的发现不计其数（论文甚至清楚提道，David Parker 和 Yann LeCun是事先发现这一研究进路的两人），1986年的这篇文章却因其精确清晰的观点陈述而显得很突出。实际上，学机器学习的人很容易发现自己论文中的描述与教科书和课堂上解释概念方式本质上相同。[IEEE 的一篇怀旧文章](http://www-isl.stanford.edu/~widrow/papers/j199030years.pdf)[^13]回应了这个观点:

> 不幸的是，科学圈里几乎无人知道Werbo的研究。1982年，Parker重新发现了这个研究办法[39]并于1985年在M.I.T[40]上发表了一篇相关报道。就在Parker报道后不久，Rumelhart, Hinton和Williams [41], [42]也重新发现了这个方法， 他们最终成功地让这个方法家喻户晓，也主要归功于陈述观点的框架非常清晰。

但是，这三位作者没有止步于介绍新学习算法，而是走得更远。同年，他们发表了更有深度的文章《[Learning internal representations by error propagation](http://psych.stanford.edu/~jlm/papers/PDP/Volume%201/Chap8_PDP86.pdf)》[^14]。文章特别谈到了Minsky在《感知机》中讨论过的问题。尽管这是过去学者的构想，但是，正是这个1986年提出的构想让人们广泛理解了应该如何训练多层神经网络解决复杂学习问题。而且神经网络也因此回来了！


在第二部分，我们将看到仅仅几年后方向传播和《Learning internal representations by error propagating》中讨论的一些技巧就被应用在解决了一个非常重大的问题上：计算机视觉（使计算机识别手写体）

[^1]: Christopher D. Manning. (2015). Computational Linguistics and Deep Learning Computational Linguistics, 41(4), 701–707.
[^2]: F. Rosenblatt. The perceptron, a perceiving and recognizing automaton Project Para. Cornell Aeronautical Laboratory, 1957.
[^3]: W. S. McCulloch and W. Pitts. A logical calculus of the ideas immanent in nervous activity. The bulletin of mathematical biophysics, 5(4):115–133, 1943.
[^4]: The organization of behavior: A neuropsychological theory. D. O. Hebb. John Wiley And Sons, Inc., New York, 1949
[^5]: B. Widrow et al. Adaptive ”Adaline” neuron using chemical ”memistors”. Number Technical Report 1553-2. Stanford Electron. Labs., Stanford, CA, October 1960.
[^6]: “New Navy Device Learns By Doing”, New York Times, July 8, 1958. 
[^7]: Perceptrons. An Introduction to Computational Geometry. MARVIN MINSKY and SEYMOUR PAPERT. M.I.T. Press, Cambridge, Mass., 1969.
[^8]: Minsky, M. (1952). A neural-analogue calculator based upon a probability model of reinforcement. Harvard University Pychological Laboratories internal report.
[^9]: Linnainmaa, S. (1970). The representation of the cumulative rounding error of an algorithm as a Taylor expansion of the local rounding errors. Master’s thesis, Univ. Helsinki.
[^10]: P. Werbos. Beyond Regression: New Tools for Prediction and Analysis in the Behavioral Sciences. PhD thesis, Harvard University, Cambridge, MA, 1974.
[^11]: Werbos, P.J. (2006). Backwards differentiation in AD and neural nets: Past links and new opportunities. In Automatic Differentiation: Applications, Theory, and Implementations, pages 15-34. Springer.
[^12]: Rumelhart, D. E., Hinton, G. E., and Williams, R. J. (1986). Learning representations by back-propagating errors. Nature, 323, 533–536. 
[^13]: Widrow, B., & Lehr, M. (1990). 30 years of adaptive neural networks: perceptron, madaline, and backpropagation. Proceedings of the IEEE, 78(9), 1415-1442.
[^14]: D. E. Rumelhart, G. E. Hinton, and R. J. Williams. 1986. Learning internal representations by error propagation. In Parallel distributed processing: explorations in the microstructure of cognition, vol. 1, David E. Rumelhart, James L. McClelland, and CORPORATE PDP Research Group (Eds.). MIT Press, Cambridge, MA, USA 318-362

# 参考

> http://www.andreykurenkov.com/writing/a-brief-history-of-neural-nets-and-deep-learning/

> https://mp.weixin.qq.com/s/iXtyLK8YHxQT09JufNF4EA##


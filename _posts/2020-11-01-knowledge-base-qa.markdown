---
layout: post
title: KBQA知识图谱问答系统及前沿探索
category: Document
tags: kbqa nlp kg ml
date: 2020-11-01 15:11:13
published: true
summary: 知识图谱问答（KBQA）利用图谱丰富的语义关联信息，能够深入理解用户问题并给出答案，近年来吸引了学术界和工业界的广泛关注。KBQA 主要任务是将自然语言问题（NLQ）通过不同方法映射到结构化的查询，并在知识图谱中获取答案。
image: pirates.svg
comment: true
---

# 知识图谱问答（KBQA）

![图一：MindMap](/postimgs/2020-11-01-kbqa.svg)

## 语义解析神经网络（Semantic Parsing Neural Network）

目前基于 Semantic Parsing 的 KBQA 主要有 4 种方法：

语义解析（Semantic Parser）过程转化为查询图生成问题的各类方法；
在领域数据集适用的 Encoder-Decoder 模型化解析方法；
基于 Transition-Based 的状态迁移学习的解析方法；
利用 KV-MemNN 进行解释性更强的深度 KBQA 模型；

### 传统方法

传统的 Semantic Parser 方法主要依赖于预先定义的规则模板，或者利用监督模型对用户 Query 和语义形式化表示（如 CCG [11]、λ-DCS [12]）的关系进行学习，然后对自然语言进行解析。

这种方法需要大量的手工标注数据，可以在某些限定领域和小规模知识库（如 ATIS [13]、GeoQuery）中达到较好的效果，但是当面临 Freebase 或 DBpedia 这类大规模知识图谱的时候，往往效果欠佳。

- CCG

  论文：Steedman M. A very short introduction to CCG[J]. Unpublished paper. http://www. coqsci. ed. ac. uk/steedman/paper. html, 1996.

- λ-DCS

  论文：Liang P. Lambda dependency-based compositional semantics[J]. arXiv preprint arXiv:1309.4408, 2013.

### 查询图方法(Query Graph)

- STAGG

  Staged Query Graph Generation.
  
  该框架首先定义了一个可以直接转化为 Lambda 演算的查询图（Query Graph），然后定义了如何将 Semantic Parser 的过程演变为查询图生成过程（4 种 State+4 种 Action）。
  
  最后通过 LambdaRank 算法对生成的查询图进行排序，选出最佳答案。查询图生成过程一共有三个主要步骤：实体链接、属性识别和约束挂载。
  
  针对 Query：“Who first voiced Meg on Family Guy?”，查询图生成过程如图 2 所示：
  
  1. 实体链接部分采用论文 [15] 的实体链接系统得到候选实体和分数；
  
  2. 属性识别阶段会根据识别的实体对候选的属性进行剪枝，并采用深度卷积模型计算 Query 和属性的相似度；
  
  3. 约束挂载阶段会根据预定义的一些规则和匹配尝试进行最值、实体约束挂载；
  
  4. 最后会根据图本身特征和每一步的分数特征训练一个 LambdaRank 模型，对候选图进行排序。
  
  此方法有效地利用了图谱信息对语义解析空间进行了裁剪，简化了语义匹配的难度，同是结合一些人工定义的处理最高级和聚合类问题的模板，具备较强理解复杂问题的能力，是 WebQuestions 数据集上强有力的 Baseline 方法。
  
  而且该方法提出的 Semantic Parser 到查询图生成转化的思想也被广泛地采纳，应用到了 Semantic Parser+NN 方法中。
  
  论文：Yih S W, Chang M W, He X, et al. Semantic parsing via staged query graph generation: Question answering with knowledge base[J]. 2015.

    - 实体链接

      论文：Yang Y, Chang M W. S-mart: Novel tree-based structured learning algorithms applied to tweet entity linking[J]. arXiv preprint arXiv:1609.08075, 2016.

    - 属性识别
    - 约束挂载

- MultiCG

  Multiple Constraint Query Graph.
  
  微软在 STAGG基础上，扩展了约束类型和算子，新增了类型约束、显式和隐式时间约束，更加系统地提出利用 MultiCG 解决复杂问题的方法。
  
  论文：Bao J, Duan N, Yan Z, et al. Constraint-based question answering with knowledge graph[C]//Proceedings of COLING 2016, the 26th International Conference on Computational Linguistics: Technical Papers. 2016: 2503-2514.

- HR-BiLSTM

  Hierarchical Residual BiLSTM:
  
  IBM 沿用 STAGG 框架，针对属性识别率较低的问题，在属性识别阶段，将 Query 和属性名称的字和词 2 个纬度进行编码，利用 HR-BiLSTM 计算相似度，提升了属性识别关键模块的准确率，该方法在 SimpleQuestions 数据集上有着优秀的表现，很好地弥补了 Semantic Gap 问题，但并没有在复杂问题的处理上进行过多的改善。
  
  论文：Yu M, Yin W, Hasan K S, et al. Improved neural relation detection for knowledge base question answering[J]. arXiv preprint arXiv:1704.06194, 2017.

- SMM

  Semantic Matching Model(SMM)
  
  分别对 Query 和查询图进行编码。
  
  首先对去掉实体的 Query 利用双向 GRU 进行全局编码得到q(tok) ，然后对 Query 进行依存分析，并利用 GRU 对依存路径进行编码，收集句法和局部语义信息，得到 q(dep)，最后两者求和得到 query 编码q(p) = q(tok) + q(dep)。
  
  然后利用查询图中的所有属性名编码和属性 id（将属性名组合）编码，得到图的编码向量 。最后利用 Cosine 距离计算 Query 与查询图的相似度
  
  该方法首次将 Query Graph 进行语义编码，计算与 Query 间的相关性程度，并以此作为匹配特征训练模型。该方法在 ComplexQuestions 数据集上的表现优于 MultiCG 的方法，但是依然不能很好地利用神经网络模型处理显隐式时间约束等复杂问题。
  
  该方法在对 Query Graph 进行编码的时候，因为最后使用了 Pooling，所以并没有考虑实体和关系的顺序问题
  
  论文：Luo K, Lin F, Luo X, et al. Knowledge base question answering via encoding of complex query graphs[C]//Proceedings of the 2018 Conference on Empirical Methods in Natural Language Processing. 2018: 2185-2194.

- Tree to Sequence

  将 Query Graph 利用 Tree-based LSTM 进行编码，虽然并没有在 WebQuestions 等数据集上取得 SOTA 的效果，但是验证了 Query 结构信息在 QA 任务上的有效性。
  
  论文：Zhu S, Cheng X, Su S. Knowledge-based question answering by tree-to-sequence learning[J].

### 编解码器方法(Encoder-Decoder)

- Seq2Seq2Tree

  爱丁堡大学提出的利用注意力机制增强的 Encoder-Decoder 模型，将 Semantic Parser 问题转化成 Seq2Seq 问题，即使用 Seq2Seq 模型完成 Query 到 Logical Forms 的转化，还提出了一个 Seq-to-Tree 模型，利用层次树形解码器捕获 Logical Form 的结构。
  
  该论文的贡献在于不依赖预定义的规则即可在领域数据集上取得 SOTA 方法，但该方法需要大量的训练预料，因此不适合通用知识问答数据集。
  
  同时通用的序列编码器利用了词序的特征而忽略了有用的句法信息。
  
  论文：https://arxiv.org/pdf/1601.01280.pdf

- Graph2Seq

  Exploiting Rich Syntactic Information for Semantic Parsing with Graph-to-Sequence Model 
  
  改论文使用句法图（Syntactic Graph）分别表示词序特征（Word Order），依存特征（Dependency）还有句子成分特征（Constituency），并采用 Graph-to-Seq 模型，利用图编码器将句法图进行编码，然后利用 RNN 对每时刻的状态向量和通过 Attention 得到的上下文信息进行解码得到对应的 Logical Forms。尽管该方法验证了额外的句法信息引入有利于提升模型的鲁棒性，但仍没有解决对于大量训练预料的依赖问题。
  
  论文：https://www.aclweb.org/anthology/D18-1110.pdf

### Transition-Based 方法

- State Transition-based

  北大提出的 State Transition-based 方法，通过定义 4 种原子操作和可学习的状态迁移模型，将复杂文图转化为 Semantic Query Graph。
  
  该方法可利用 BiLSTM 模型识别出多个节点（如多个实体），并用 Multi-Channel Convolutional  Neural Network（MCCNN）抽取节点间的多个关系，克服了查询图生成方法中假设只有一个主要关系和利用人工规则识别约束的缺点，具备更强大的鲁棒性。
  
  同时该方法的状态迁移是个可学习的过程，而此前的查询图生成方法中的状态迁移过程是预定义的。
  
  论文：https://www.aclweb.org/anthology/D18-1234.pdf

    - 4种原子操作
    - 可学习状态迁移模型
    - Semantic Query Graph

- Stack-LSTM

  论文：https://arxiv.org/pdf/1704.08387v2.pdf

### Memory Network 方法

论文：https://arxiv.org/pdf/1410.3916.pdf

- KV-MemNN

  Key-Value Memory Network（KV-MemNN），此结构相比于扁平的 Memory Network 结构，可以很好地存储具有复杂结构的数据（如 KG），并通过在多个记忆槽内的预测来进行浅层的多跳推理任务，最终生成相关子图路径。
  
  论文：https://arxiv.org/pdf/1606.03126.pdf

- Enhanced KV-MemNN

  该论文思路：
  
  用 KV-MemNN 来存储图谱中的三元组信息
  新的 Query Update 机制，在更新时去掉 Query 中已经定位到的 Key 值，可以让模型能更好地注意到下一步需要推理的内容
  Key Hashing 阶段加入特殊的 STOP 键值，来告诉模型在时刻 t 是否收集来足够的信息来回答用户 Query，实验验证来它能避免复杂问题无效的推理过程。
  
  在预测阶段，验证来利用 Semantic Parser 的结果要明显优于 IR 方法，作者认为 Semantic Parser 方法在每一步预测中都会选择最优的 key，比 IR 方法（只用最后一步的表示输出进行排序）利用来更多全局的信息。
  
  该方法在不依赖于手工构造模板的情况下，在 WebQuestions 数据集上的效果超过来 SOTA 方法，而且该方法的框架引入的 STOP、Query Update 机制让整个复杂问题的解析过程具备更好的解释性。
  
  这种利用 KG 进行层次化的解析方法是 KBQA 发展的趋势之一。
  
  论文：https://pdfs.semanticscholar.org/8bc3/3f2d6bb9e6100b720e09c44e6c3ebee5614f.pdf

## 信息检索（Information Retrieval）

基于搜索排序（IR）的知识图谱问答的工作流：
首先会确定用户 Query 中实体提及词（Entity Mention）；
然后链接到 KG 中的主题实体（Topic Entity），并将 Topic Entity 相关的子图（Subgraph）提取出来作为候选答案集合；
然后分别从 Query 和候选答案中抽取特征；
最后利用排序模型对 Query 和候选答案进行建模并预测

此类方法不需要大量人工定义特征或者模板，将复杂语义解析问题转化为大规模可学习问题。

依据特征表示技术不同，分为基于特征工程的方法和基于表示学习的方法。

### 基于特征工程的方法

《Information Extraction over Structured Data: Question Answering with Freebase 》是该类方法的基础模型，处理流程：

先对问句进行句法分析，并对其依存句法分析结果提取问题词（qword）、问题焦点词（qfocus）、主题词（qtopic）和中心动词（qverb）特征，将其转化为问句特征图（Question Graph）
然后利用 qtopic 在 KG 内提取 Subgraph，并基于此生成候选答案特征图
最后将问句中的特征与候选特征图中的特征进行组合，将关联度高的特征赋予较高的权重，该权重的学习直接通过分类器学习

但这种方法有两点不足：
需要自行定义并抽取特征，而且问句特征和候选答案特征组合需要进行笛卡尔乘积，特征纬度过大；
此方法难以处理复杂问题；

论文：http://cs.jhu.edu/~xuchen/paper/yao-jacana-freebase-acl2014.pdf

### 基于表示学习的方法

鉴于基于特征工程的两点不足，基于表示学习的方法进行了改进

- Embedding-Based 方法

  主要工作流：
  将问句以及 KG 中的候选答案实体映射到同一语义空间，其中候选答案实体利用三种向量进行表示：1）答案本身；2）答案实体与主实体关系路径；3）与答案实体相关 Subgraph
  然后利用 Triplet loss1 对模型进行训练
  
  该模型针对问句编码采用词袋模型，没有考虑词序对句子的影响，以及不同类型属性的不同特征。
  
  论文：https://arxiv.org/pdf/1406.3676.pdf

- CNN and Attention 方法

  Embedding-Based 方法针对问句编码采用词袋模型，没有考虑词序对句子的影响，以及不同类型属性的不同特征。
  
  所以 CNN and Attention 方法将更多词序等的特征引入进来。

    - MCCNNs

      Multi-Column Convolutional Neural Networks（MCCNNs），论文思路：
      
      利用 CNN 分别对问句和答案类型（Answer Type）、答案实体关系路径（Answer Path）和答案实体一跳内的 Subgraph（Answer Context）进行编码，以获取不同语义表示。
      
      该方法验证了考虑词序信息、问句与答案的关系对 KBQA 效果的提升是有效的。
      
      但该方法对于不同答案，将问句都转换成一个固定长度的向量，不能很好地表达问句的信息。
      
      论文：https://www.aclweb.org/anthology/P15-1026.pdf

    - CANN

      Cross-Attention based Neural Network：
      针对 MCCNNs 的问题，CANN 的思路：
      
      将候选答案分成四个维度：答案实体 $\alpha_e$、答案实体关系路径 $\alpha_r$、答案类型 $\alpha_t$ 和答案上下文 $\alpha_c$，然后利用注意力机制动态学习不同答案纬度与问句中词的关联程度，让问句对不同答案纬度根据注意力机制学习权重，有效的提升问答效果。

    - AR-SMCNN

      Attentive RNN with Similarity Matrix based CNN(AR-SMCNN)，论文思路：
      
      利用带有注意力机制的 RNN 来对问句和属性间关系进行建模，并采用 CNN 模型捕获问句与属性间的字面匹配信息。
      
      该方法验证了字面匹配信息能带来效果上的提升，而实验结果显示，以上两种网络模型对于复杂问题的处理能力依然不足。
      
      论文地址：https://arxiv.org/vc/arxiv/papers/1804/1804.03317v2.pdf

- Memory Network 方法

  与 Semantic Parser 方法相同，Memory Network（MemNN）有其良好的可扩展性。

    - WSMNN

      Weakly Supervised Memory Networks（WSMN）
      
      若监督适用性
      
      论文地址：https://arxiv.org/pdf/1503.08895v2.pdf

    - LsSQAMNN

      Large-scale Simple Question Answering with Memory Networks 
      
      论文思路：用 MemNN结合 KG 中三元组信息来解决 KBQA 中简单问题（单跳）。
      
      将 KG 中的知识存储到 MemNN 中。通过输入模块（Input module）将用户问题、三元组信息转化成分布式表示加入到 Memory 中；
      然后利用泛化模块（Generalization module）将新的三元组信息（来自 Reverb）加入到 Memory 中；
      然后采样正负例 Query 和正负例的 Answer，两两组合成多任务，利用 Triplet loss 训练 MemNN
      输出模块（Output module）从 Memory 中选择一些与问题相关性高的三元组信息；
      回复模块（Response module）返回从输出模块中得到的答案
      
      此论文为后续的 KBQA 研究奠定来基础
      
      论文地址：https://arxiv.org/pdf/1506.02075.pdf

    - L-FMNN

      论文提出 一个 L 层的 Factual Memory Network 来模拟多跳推理过程，每层以线性链接，并以上一层输出作为本层输入。
      
      第一层的输入是候选三元组和问句。对问句编码时，为弥补词袋模型无法考虑词序的问题，为每一个词 $x_i$ 的不同纬度 $j$ 设计一种位置编码（Position Encoding）
      
      $l_{ij} = \min(\frac{i*d}{j*|q|}, \frac{j*|q|}{i*d})$
      
      其中 $d$ 是词向量纬度。最后使用位置编码更新句向量 $q = \sum_{i=1}^{|q|} l_i \odot x_i$。这应该是首次利用深度模型结合 KG 信息层次化地进行复杂问题的语义解析工作，不仅在 WebQuestions 上验证来这种结构的有效性，也为深度学习模型带来了更好地解释性。
      
      尽管如此，在编码阶段，将问句和 KG 三元组信息分别进行映射，没有考虑两者之间的交互关系。
      
      论文地址：https://www.aclweb.org/anthology/N16-2016.pdf

    - BAMnet

      Bidirectional Attentive Memory Network(BAMnet) 模型，主要通过 Attention 机制来捕获 Query 与 KG 信息间的两两相关性，并以此利用相关性增强 Query 的表示，希望能结合到更多的 KG 信息，提升针对复杂问题的处理能力，该方法包括 4 个模块：
      输入模块采用 Bi-LSTM 对问句进行编码；
      记忆模块（Memory module）针对 KG 信息（答案类型 $h_t$、答案实体关系路径 $h_p$、答案上下文 $h_c$）进行编码存入 Memory network 中
      推理模块分为一个两层的 Bidirectional Attention Network 去编码问句和 KG 中信息间的相互关系和泛化模块（Generalization module）模型中 Primary attention network 中的 KB-aware attention module 会根据 KG 信息关注问句中的重要部分；Importance module 利用注意力机制计算 KG 中信息与问句的相关程度（比如简单问题对于答案上下文关注度就会降低）；然后在 Enhancing module 中利用 Importance module 计算的相关度权重矩阵，重新更新问句表示和 Memory 中的 Key 值表示，完成第二层 Attention Network；最后在泛化模块中选取 Memory 中对与问句表示有价值 value，利用 GRU 进行编码，链接到原有问句表示； 
      回复模块（Answer module）通过计算问句信息和 Enhancing module 中更新过的 KG 信息来得到最后的答案
      
      该方法与众多 IR 方法一样，在不依赖于人工构建模板的情况下，依靠 Attention 机制和 MemNN，效果比大多数 IR 方法更好，但是并没有引用了 PE 机制的论文效果好，我们猜测如果利用 Transformer 的思想，引入词语位置信息，也许可以取得不错的效果。
      
      论文地址：https://arxiv.org/abs/1903.02188

## 其他方法

### 从 KG/QA 中学习模版，然后将问句拆解成 Logical Forms 或理解其意图

### 把复杂问句分解成几个简单问句，然后从多个简单答案中找出最终的复杂问句答案

### Message Parsing NN + Stepwise Reasoning Network

### Neural Symbolic Machine(NSM)

## 开放数据集

### WebQuestions

### SimpleQuestions

### ComplexQuestions

## 论文集

[1] Yang Z, Qi P, Zhang S, et al. Hotpotqa: A dataset for diverse, explainable multi-hop question answering[J]. arXiv preprint arXiv:1809.09600, 2018.
[2] Saha A, Pahuja V, Khapra M M, et al. Complex sequential question answering: Towards learning to converse over linked question answer pairs with a knowledge graph[C]//Thirty-Second AAAI Conference on Artificial Intelligence. 2018.
[3] Berant J, Chou A, Frostig R, et al. Semantic parsing on freebase from question-answer pairs[C]//Proceedings of the 2013 conference on empirical methods in natural language processing. 2013: 1533-1544.
[4] Yih W, Richardson M, Meek C, et al. The value of semantic parse labeling for knowledge base question answering[C]//Proceedings of the 54th Annual Meeting of the Association for Computational Linguistics (Volume 2: Short Papers). 2016: 201-206.
[5] Bao J, Duan N, Yan Z, et al. Constraint-based question answering with knowledge graph[C]//Proceedings of COLING 2016, the 26th International Conference on Computational Linguistics: Technical Papers. 2016: 2503-2514.
[6] 刘康,面向知识图谱的问答系统. AI前沿讲习班第8期PPT
[7] 赵军, 刘康, 何世柱, 陈玉博. 知识图谱[B]. 2018.
[8] Trivedi P, Maheshwari G, Dubey M, et al. Lc-quad: A corpus for complex question answering over knowledge graphs[C]//International Semantic Web Conference. Springer, Cham, 2017: 210-218.
[9] Dubey M, Banerjee D, Abdelkawi A, et al. Lc-quad 2.0: A large dataset for complex question answering over wikidata and dbpedia[C]//International Semantic Web Conference. Springer, Cham, 2019: 69-78.
[10] Liang C, Berant J, Le Q, et al. Neural symbolic machines: Learning semantic parsers on freebase with weak supervision[J]. arXiv preprint arXiv:1611.00020, 2016.
[11] Steedman M. A very short introduction to CCG[J]. Unpublished paper. http://www. coqsci. ed. ac. uk/steedman/paper. html, 1996.
[12] Liang P. Lambda dependency-based compositional semantics[J]. arXiv preprint arXiv:1309.4408, 2013.
[13] Dahl D A, Bates M, Brown M, et al. Expanding the scope of the ATIS task: The ATIS-3 corpus[C]//Proceedings of the workshop on Human Language Technology. Association for Computational Linguistics, 1994: 43-48.
[14] Yih S W, Chang M W, He X, et al. Semantic parsing via staged query graph generation: Question answering with knowledge base[J]. 2015.
[15]  Yang Y, Chang M W. S-mart: Novel tree-based structured learning algorithms applied to tweet entity linking[J]. arXiv preprint arXiv:1609.08075, 2016.
[16] Yu M, Yin W, Hasan K S, et al. Improved neural relation detection for knowledge base question answering[J]. arXiv preprint arXiv:1704.06194, 2017.
[17] Luo K, Lin F, Luo X, et al. Knowledge base question answering via encoding of complex query graphs[C]//Proceedings of the 2018 Conference on Empirical Methods in Natural Language Processing. 2018: 2185-2194.
[18] Zhu S, Cheng X, Su S. Knowledge-based question answering by tree-to-sequence learning[J]. Neurocomputing, 2020, 372: 64-72.
[19] Wu P, Zhang X, Feng Z. A Survey of Question Answering over Knowledge Base[C]//China Conference on Knowledge Graph and Semantic Computing. Springer, Singapore, 2019: 86-97.
[20] Dong L, Lapata M. Language to logical form with neural attention[J]. arXiv preprint arXiv:1601.01280, 2016.
[21] Xu K, Wu L, Wang Z, et al. Exploiting rich syntactic information for semantic parsing with graph-to-sequence model[J]. arXiv preprint arXiv:1808.07624, 2018.
[22] Hu S, Zou L, Zhang X. A state-transition framework to answer complex questions over knowledge base[C]//Proceedings of the 2018 Conference on Empirical Methods in Natural Language Processing. 2018: 2098-2108.
[23] Cheng J, Reddy S, Saraswat V, et al. Learning structured natural language representations for semantic parsing[J]. arXiv preprint arXiv:1704.08387, 2017.
[24] Dyer C, Ballesteros M, Ling W, et al. Transition-based dependency parsing with stack long short-term memory[J]. arXiv preprint arXiv:1505.08075, 2015.
[25] Weston J, Chopra S, Bordes A. Memory networks[J]. arXiv preprint arXiv:1410.3916, 2014.
[26] Miller A, Fisch A, Dodge J, et al. Key-value memory networks for directly reading documents[J]. arXiv preprint arXiv:1606.03126, 2016.
[27] Xu K, Lai Y, Feng Y, et al. Enhancing Key-Value Memory Neural Networks for Knowledge Based Question Answering[C]//Proceedings of the 2019 Conference of the North American Chapter of the Association for Computational Linguistics: Human Language Technologies, Volume 1 (Long and Short Papers). 2019: 2937-2947.
[28] Yao X, Van Durme B. Information extraction over structured data: Question answering with freebase[C]//Proceedings of the 52nd Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers). 2014: 956-966.
[29] Bordes A, Chopra S, Weston J. Question answering with subgraph embeddings[J]. arXiv preprint arXiv:1406.3676, 2014.
[30] Dong L, Wei F, Zhou M, et al. Question answering over freebase with multi-column convolutional neural networks[C]//Proceedings of the 53rd Annual Meeting of the Association for Computational Linguistics and the 7th International Joint Conference on Natural Language Processing (Volume 1: Long Papers). 2015: 260-269.
[31] Hao Y, Zhang Y, Liu K, et al. An end-to-end model for question answering over knowledge base with cross-attention combining global knowledge[C]//Proceedings of the 55th Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers). 2017: 221-231.
[32] Qu Y, Liu J, Kang L, et al. Question answering over freebase via attentive RNN with similarity matrix based CNN[J]. arXiv preprint arXiv:1804.03317, 2018, 38.
[33] Sukhbaatar S, Szlam A, Weston J, et al. Weakly supervised memory networks[J]. CoRR, abs/1503.08895, 2.
[34] Bordes A, Usunier N, Chopra S, et al. Large-scale simple question answering with memory networks[J]. arXiv preprint arXiv:1506.02075, 2015.
[35]  Jain S. Question answering over knowledge base using factual memory networks[C]//Proceedings of the NAACL Student Research Workshop. 2016: 109-115.
[36] Chen Y, Wu L, Zaki M J. Bidirectional attentive memory networks for question answering over knowledge bases[J]. arXiv preprint arXiv:1903.02188, 2019.
[37] Vaswani A, Shazeer N, Parmar N, et al. Attention is all you need[C]//Advances in neural information processing systems. 2017: 5998-6008.
[38] Reddy S, T_ckstr_m O, Collins M, et al. Transforming dependency structures to logical forms for semantic parsing[J]. Transactions of the Association for Computational Linguistics, 2016, 4: 127-140.
[39] Abujabal A, Yahya M, Riedewald M, et al. Automated template generation for question answering over knowledge graphs[C]//Proceedings of the 26th international conference on world wide web. 2017: 1191-1200.
[40] Cui W, Xiao Y, Wang H, et al. KBQA: learning question answering over QA corpora and knowledge bases[J]. arXiv preprint arXiv:1903.02419, 2019.
[41] Talmor A, Berant J. The web as a knowledge-base for answering complex questions[J]. arXiv preprint arXiv:1803.06643, 2018.
[42] Vakulenko S, Fernandez Garcia J D, Polleres A, et al. Message Passing for Complex Question Answering over Knowledge Graphs[C]//Proceedings of the 28th ACM International Conference on Information and Knowledge Management. 2019: 1431-1440.
[43] Qiu Y, Wang Y, Jin X, et al. Stepwise Reasoning for Multi-Relation Question Answering over Knowledge Graph with Weak Supervision[C]//Proceedings of the 13th International Conference on Web Search and Data Mining. 2020: 474-482.
[44] Petrochuk M, Zettlemoyer L. Simplequestions nearly solved: A new upperbound and baseline approach[J]. arXiv preprint arXiv:1804.08798, 2018.
[45] Zhang J, Ye Y, Zhang Y, et al. Multi-Point Semantic Representation for Intent Classification[J]. AAAI. 2020.
[46] Jia R, Liang P. Adversarial examples for evaluating reading comprehension systems[J]. arXiv preprint arXiv:1707.07328, 2017.
[47] Chakraborty N, Lukovnikov D, Maheshwari G, et al. Introduction to Neural Network based Approaches for Question Answering over Knowledge Graphs[J]. arXiv preprint arXiv:1907.09361, 2019.
[48] Ribeiro M T, Singh S, Guestrin C. Semantically equivalent adversarial rules for debugging nlp models[C]//Proceedings of the 56th Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers). 2018: 856-865.


> https://mp.weixin.qq.com/s/NkwyHsmpZCwsQPiQOMK7zw


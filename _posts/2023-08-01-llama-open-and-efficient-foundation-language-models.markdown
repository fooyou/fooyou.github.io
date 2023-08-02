---
layout: post
title: 【论文】LLaMA：开放和搞笑的基础语言模型集
category: Document
tags: llm llama
date: 2023-08-01 10:08:02
published: true
summary: 
cover: 
comment: false
---

作者：Hugo Touvron∗, Thibaut Lavril∗, Gautier Izacard∗, Xavier Martinet, Marie-Anne Lachaux, Timothee Lacroix, Baptiste Rozière, Naman Goyal, Eric Hambro, Faisal Azhar, Aurelien Rodriguez, Armand Joulin, Edouard Grave∗, Guillaume Lample∗

机构：Meta AI

## 摘要

本文介绍 LLaMA，一个包含 7B~65B（70~650 亿） 参数的基础语言模型集（a collection of foundation language models）。 我们用数万亿个（trillions of） token 训练这些模型，证明了使用公开数据集就能训练出最先进的模型， 而并非必须使用专有和私有数据集。特别是，LLaMA-13B 在大多数基准测试中优于 GPT-3（175B） ，而 LLaMA-65B 则与最佳模型 Chinchilla-70B 和 PaLM-540B 相当。 我们已经将所有模型开源，供社区研究。

## 1.引言

在大规模文本语料库（massive corpora of texts）上训练的大型语言模型 （Large Languages Models, LLM），已经有能力根据给定的文本指令（textual instructions） 或示例（a few examples）执行新任务（Brown 等，2020）。

这些 few-shot 属性首先出现在将模型扩展到足够大的规模时（Kaplan 等，2020）， 在此之后，出现了很多进一步扩展这些模型的工作（Chowdhery 等，2022；Rae 等，2021）， 它们都遵循了这样一个假设：更多的参数将产生更好的性能。 然而，Hoffmann 等（2022）的最新工作表明，对于给定的计算预算（compute budget）， 最佳性能并非来自那些最大的模型，而是来自那些在更多数据上训练出来的较小模型。

### 1.1 大模型训练：更多参数 vs 更大的数据集

Hoffmann 等（2022）的提出了 scaling laws，目标是针对给定的训练（training） 计算预算（compute budget），如何最佳地扩展（scale）数据集和模型大小。但是，

- 这个模型没有考虑推理（inference）预算，在提供大规模推理时，这一点尤其重要： 在这种情况下，给定一个性能目标，我们更想要的是一个推理速度最快而非训练速度最快的模型。
- 对于一个给定的性能要求，训练一个大模型（a large model）可能是一种更便宜的方式； 但对于最终的推理来说，较小的模型+更长的训练时间（a smaller one trained longer）反而更实惠。 例如，Hoffmann 等（2022）建议用 200B tokens 来训练 10B 模型，但我们发现即使在 1T 个 token 之后，7B 模型的性能仍在随着 token 的增多而提高。

### 1.2 LLaMA：减少参数，增大数据集

本文的重点是：对于给定的不同推理预算（inference budgets）， 通过使用更多 token 进行训练的方式（超过业内常用的 token 规模）， 来获得最佳的性能（the best possible performance）。 由此得到的模型我们称为 LLaMA。 LLaMA 的参数范围在 7B ~ 65B，性能则与目前业界最佳的一些大语言模型（LLM）相当。 例如，

- LLaMA-13B 在大多数基准测试中优于 GPT-3， 尽管参数连后者的 10% 都不到。LLaMA 可以在单个 GPU 上运行， 因此使大模型的获取和研究更容易，而不再只是少数几个大厂的专利；
- 在高端系列上，LLaMA-65B 也与最佳的大语言模型（如 Chinchilla 或 PaLM-540B）性能相当。

与 Chinchilla、PaLM、GPT-3 不同，我们只使用公开数据（publicly available data）， 因此我们的工作是开源兼容的；

- 相比之下，大多数现有模型依赖于不公开或没有文档的数据（not publicly available or undocumented），例如 “Books–2TB” 和 “Social media conversations”；
- 也存在一些例外，例如 OPT（Zhang 等，2022）、GPT-NeoX（Black 等，2022）、BLOOM（Scao 等，2022）和 GLM（Zeng 等，2022）， 但它们的性能都无法与 PaLM-62B 或 Chinchilla 相比。

### 1.3 内容组织

本文接下来的内容组织如下：

- 描述我们对 Transformer 架构（Vaswani 等，2017）所做的改动，以及我们的训练方法:
- 给出 LLaMA 的性能，基于标准基准测试与其他 LLM 进行比较；
- 使用 responsible AI 社区的最新基准测试，揭示 LLaMA 模型中存在的一些偏见和毒性（biases and toxicity）。

## 2.方法（Approach）

我们的训练方法与前人的一些工作（Brown 等，2020；Chowdhery 等，2022）类似， 并受到 Chinchilla scaling laws（Hoffmann 等，2022）的启发。 我们使用一个标准的 optimizer 在大量文本数据上训练大型 Transformers。

### 2.1 预训练数据（Pre-training Data）

#### 2.1.1 数据集

我们的训练数据集有几种不同来源，涵盖了多个领域，如表 1 所示。

 数据集        | 占比  | 迭代次数（epoches）  | 数据集大小（Disk size）
---------------|-------|----------------------|-----------------------------
 CommonCrawl   | 67.0% | 1.10                 | 3.3 TB
 C4            | 15.0% | 1.06                 | 783 GB
 Github        |  4.5% | 0.64                 | 328 GB
 Wikipedia     |  4.5% | 2.45                 |  83 GB
 Books         |  4.5% | 2.23                 |  85 GB
 ArXiv         |  2.5% | 1.06                 |  92 GB
 StackExchage  |  2.0% | 1.03                 |  78 GB

_表1：**预训练数据。**其中 epochs 是用 1.4T tokens 预训练时的迭代次数。用 1T tokens 预训练时也是用的这个数据集比例。_

##### English CommonCrawl [67%]

我们使用 CCNet pipeline（Wenzek 等，2020）对 2017~2020 的五个 CommonCrawl dumps 进行预处理。

- 在行级别（line level）上对数据去重，
- 使用 fastText 线性分类器进行语言识别，去掉非英文网页，
- 使用 ngram 语言模型过滤掉一些低质量内容。

此外，我们还训练了一个线性模型，将页面分为两类：

- 被 Wikipedia 引用过的网页；
- 没有被 Wikipedia 引用过的（随机采样网页）；

并将第二类丢弃。

##### C4 [15%]

在前期探索性实验中，我们观察到使用多样化的预处理 CommonCrawl 数据集可以提高性能。 因此，我们将公开可用的 C4 数据集（Raffel 等，2020）也包含到了训练数据中。

对 C4 的预处理也是去重和语言识别：与 CCNet 的主要区别在于质量过滤（quality filtering）， 主要依赖于启发式方法（heuristics），例如是否存在标点符号或网页中单词和句子的数量。

##### Github [4.5%]

使用了 Google BigQuery 上公开可用的 GitHub 数据集，但仅保留其中用 Apache、BSD 和 MIT license 的项目。 此外，

- 基于行长度（line length），字母或数字字符（alphanumeric characters）比例等，用启发式方法过滤掉低质量文件；
- 使用正则表达式删除一些模板段落（boilerplate），例如 headers；
- 在文件级别上使用精确匹配对得到的数据集进行去重。

##### Wikipedia [4.5%]

使用了 2022 年 6 月至 8 月的一部分 Wikipedia dumps， 覆盖 20 种语言（use either the Latin or Cyrillic scripts）：bg、ca、cs、da、de、en、es、fr、hr、hu、it、nl、pl、pt、ro、ru、sl、sr、sv、uk。

删掉了其中的超链接、注释和其他 formatting boilerplate。

##### Gutenberg and Books3 [4.5%]

训练数据集中包含两个书籍语料库：

1. Gutenberg Project：公版书（public domain books）；
2. Books3 section of ThePile（Gao 等，2020）：一个用于训练大语言模型的公开可用数据集。

在书级别（book level）去重，内容超过 90% 重复的书会被剔除出去。

##### ArXiv [2.5%]

为了让训练数据集包含一定的科学数据（scientific data），我们对一些 arXiv Latex 文件做处理之后加到训练数据集。

- 按照 Lewkowycz 等（2022）的方法，删除了 the first section 之前的所有内容以及参考文献，
- 从 .tex 文件中删除了注释，
- 对作者编写的定义和宏（definitions and macros written by users）做了内联展开（inline-expand），使得论文更加一致（increase consistency across papers）。

##### Stack Exchange [2%]

Stack Exchange 是一个高质量的问答网站，涵盖了从计算机科学到化学等各种领域。 我们的训练数据集包括了一个 Stack Exchange dump，

- 保留其中最大的 28 个网站的数据，
- 从文本中删除了 HTML tags ，
- 按分数（从高到低）对答案进行了排序。

#### 2.1.2 Tokenizer（分词器）

我们使用 bytepair encoding（BPE）算法（Sennrich 等，2015）对数据进行 tokenization，算法实现采用的是 Sentence-Piece（Kudo 和 Richardson，2018）。需要说明的是，为了 decompose unknown UTF-8 characters，我们将所有 numbers 拆分为单个 digits，再 fallback 到 bytes。

最终，我们的整个训练数据集在 tokenization 后包含大约 1.4T 个 token。

- 对于大多数训练数据，每个 token 在训练期间仅使用一次；
- 维基百科和书籍是个例外，会被使用两次（two epochs）。

### 2.2 架构（Architecture）

与最近大语言模型的研究趋势一致，我们的网络也基于 Transformer 架构（Vaswani 等，2017）。 但我们做了很多改进，也借鉴了其他模型（例如 PaLM）中的一些技巧。

#### 2.2.1 改进

以下是与原始架构的主要差异，

##### 预归一化（Pre-normalization）：灵感来自 GPT3

为了提高训练稳定性，我们对每个 Transformer sub-layer 的输入进行归一化，而不是对输出进行归一化。 这里使用由 Zhang 和 Sennrich（2019）提出的 RMSNorm 归一化函数。

##### SwiGLU 激活函数：灵感来自 PaLM

用 SwiGLU 激活函数替换 ReLU 非线性，该函数由 Shazeer（2020）提出，目的是提升性能。 但我们使用的维度是 2/3 * 4d，而不是 PaLM 中的 4d。

##### 旋转嵌入（Rotary Embeddings）：灵感来自 GPTNeo

去掉了绝对位置嵌入（absolute positional embeddings），并在每个网络层中添加旋转位置嵌入（rotary positional embeddings，RoPE）。 RoPE 由 Su 等（2021）提出。

#### 2.2.2 不同 LLaMA 模型的超参数

 params | dimension | n heads | n layers | learning rate | batch size | n tokens
--------|-----------|---------|----------|---------------|------------|----------------
 6.7B   | 4096      | 32      | 32       | 3.0e-4        | 4M         | 1.0T
 13.0B  | 5120      | 40      | 40       | 3.0e-4        | 4M         | 1.0T
 32.5B  | 6656      | 52      | 60       | 1.5e-4        | 4M         | 1.4T
 65.2B  | 8192      | 64      | 80       | 1.5e-4        | 4M         | 1.4T

*表2：Model sizes, architectures, and optimization hyper-parameters.*

### 2.3 优化器（Optimizer）

- 使用 AdamW 优化器（Loshchilov 和 Hutter，2017）对模型进行训练，具体超参数：β1=0.9,β2=0.95；
- 使用一个 cosine learning rate schedule，最终的学习率达到了最大学习率的 10％；
- 使用 0.1 的权重衰减（weight decay）和 1.0 的梯度裁剪（gradient clipping）；
- 使用 2,000 个 warmup steps，并根据模型大小来调整 learning rate 和 batch size。

![figure 1: Training loss over train tokens for the 7B, 13B, and 65B models](/postimgs/llama1_training_loss.jpg)

*图1：7B, 13B, 33B, 和 65B 模型的训练损失。LLaMA-33B 和 LLaMA-65B 是在 1.4T token 上训练的，小点的模型是在 1.0T token 上训练的，所有模型的训练 batch 大小为 4M token*

### 2.4 高效实现（Efficient implementation）：提高训练速度

我们进行了几项优化来提高模型的训练速度。

首先，我们使用 causal multi-head attention 的一个高效实现来减少内存占用和运行时。 这种实现是受 Rabe 和 Staats（2021）的启发，并使用 Dao 等（2022）的反向传播，现在 xformers 库 中已经提供了。 优化原理：由于语言建模任务存在因果特性，因此可以不存储注意力权重（attention weights），不计算那些已经被掩码（masked）的 key/query scores。

为进一步提高训练效率，我们通过 checkpoint 技术， 减少了在反向传播期间需要重新计算的激活数量。更具体地说，

- 我们保存了计算成本高昂的激活，例如线性层的输出。实现方式是手动实现 Transformer 层的反向函数，而不用 PyTorch autograd。
- 如 Korthikanti 等（2022）中提到的， 为了充分受益于这种优化，我们需要通过模型和序列并行（model and sequence parallelism）来减少模型的内存使用。
- 此外，我们还尽可能地 overlap 激活计算和 GPU 之间的网络通信（由于 `all_reduce` 操作）。

训练 65B 参数的模型时，我们的代码在 2048 个 A100 80GB GPU 上能处理约 `380 tokens/second/GPU`。这意味着 1.4T token 的数据集上训练大约需要 21 天。

## 3.主要结果（Main results）

参考前人工作（Brown 等，2020），我们测试了零样本（zero-shot）和少样本（few-shot）两种任务， 进行总共 20 个基准测试：

- 零样本：提供任务的文本描述和一个测试示例。模型可以使用开放式生成（open-ended generation）提供答案，或对提议的答案进行排名（ranks the proposed answers）。
- 少样本：提供一些（1~64 个）任务示例和一个测试示例。模型将此文本作为输入并生成答案，或对不同选项进行排名（ranks different options）。

我们将 LLaMA 与其他基础模型进行比较，包括

- 未开源模型（non-publicly available）：GPT-3（Brown 等，2020）、Gopher（Rae 等，2021）、Chinchilla（Hoffmann 等，2022）和 PaLM（Chowdhery 等，2022），
- 开源模型：OPT 模型（Zhang 等，2022）、GPT-J（Wang 和 Komatsuzaki，2021）和 GPTNeo（Black 等，2022）。
- 在第 4 节中，我们还将简要比较 LLaMA 与 instruction-tuned 模型，如 OPT-IML（Iyer 等，2022）和 Flan-PaLM（Chung 等，2022）。

我们在自由形式生成任务（free-form generation）和多项选择（multiple choice）任务上评估 LLaMA。 多项选择任务的目标是在提供的上下文基础上，从一组给定选项中选择最合适的。我们使用的最合适标准就是可性能最高（highest likelihood）。

- 对于大部分数据集，我们遵循 Gao 等（2021）的方法，使用由完成字符数归一化的可能性（likelihood normalized by the number of characters），
- 对于少量数据集（OpenBookQA，BoolQ），我们遵循 Brown 等（2020）的方法，根据在“Answer:”上下文中给定的完成可能性（likelihood of the completion given “Answer:” as context），用公式表示就是 `P(completion|context) / P(completion|"Answer:")`.

### 3.1 常识推理（Common Sense Reasoning）

使用下面八个标准的常识推理基准测试：

- BoolQ（Clark 等，2019）
- PIQA（Bisk 等，2020）
- SIQA（Sap 等，2019）
- HellaSwag（Zellers 等，2019）
- WinoGrande（Sakaguchi 等，2021）
- OpenBookQA（Mihaylov 等，2018）
- & 8. ARC easy 和 challenge（Clark 等，2018）

这些数据集包括 Cloze 和 Winograd 风格的任务，以及多项选择题。 与语言建模社区类似，我们使用零样本设置进行评估。 在表 3 中，我们与各种规模的现有模型进行比较。

 Model      | psz  | BoolQ | PIQA | SIQA | HellaSwag | WinoGrande | ARC-e | ARC-c | OBQA
------------|------|-------|------|------|-----------|------------|-------|-------|---------------
 GPT-3      | 175B | 60.5  | 81.0 | -    | 78.9      | 70.2       | 68.8  | 51.4  | 57.6
 Gopher     | 280B | 79.3  | 81.8 | 50.6 | 79.2      | 70.1       | -     | -     | -
 Chinchilla | 70B  | 83.7  | 81.8 | 51.3 | 80.8      | 74.9       | -     | -     | -
 PaLM       | 62B  | 84.8  | 80.5 | -    | 79.7      | 77.0       | 75.2  | 52.5  | 50.4
 PaLM-cont  | 62B  | 83.9  | 81.4 | -    | 80.6      | 77.0       | -     | -     | -
 PaLMa      | 540B |**88.0**|82.3 | -    | 83.4      |**81.1**    | 76.6  | 53.0  | 53.4
 LLaMA      | 7B   | 76.5  | 79.8 | 48.9 | 76.1      | 70.1       | 72.8  | 47.6  | 57.2
 LLaMA      | 13B  | 78.1  | 80.1 | 50.4 | 79.2      | 73.0       | 74.8  | 52.7  | 56.4
 LLaMA      | 33B  | 83.1  | 82.3 | 50.4 | 82.8      | 76.0       |**80.0**|**57.8**| 58.6
 LLaMA      | 65B  | 85.3  |**82.8**|**52.3**|**84.2**|77.0       | 78.9  | 56.0  |**60.2**

*表3：Zero-shot performance on Common Sense Reasoning tasks*

几点说明：

- 除了 BoolQ，LLaMA-65B 在其他所有基准测试都优于 Chinchilla-70B。
- 同样，该模型在除了 BoolQ 和 WinoGrande 之外的所有地方都超过了 PaLM-540B。
- LLaMA-13B 模型尽管比 GPT-3 小 90％ 多，但在大多数基准测试中表现比 GPT-3 还好。

### 3.2 闭卷问答（Closed-book Question Answering）

我们将 LLaMA 与现有的大语言模型进行比较，在两个闭卷问答基准测试：

- 自然问题（Kwiatkowski 等，2019）
- TriviaQA（Joshi 等，2017）。

对于这两个基准测试，在相同设置（例如，模型不能访问那些有助于回答问题的文档）下， 取得了完全相同的性能（exact match performance）。 表 4 和表 5 分别是在这两个 benchmark 上的结果，

 Model      | psz  | 0-shot | 1-shot | 5-shot | 64-shot
------------|------|--------|--------|--------|----------
 GPT-3      | 175B | 14.6   | 23.0   |    -   | 29.9
 Gopher     | 280B | 10.1   |    -   | 24.5   | 28.2
 Chinchilla |  70B | 16.6   |    -   | 31.5   | 35.5
 PaLM       |   8B |  8.4   | 10.6   |    -   | 14.6
 PaLM       |  62B | 18.1   | 26.5   |    -   | 27.6
 PaLM       | 540B | 21.2   | 29.3   |    -   | 39.6
 LLaMA      |   7B | 16.8   | 18.7   | 22.0   | 26.1
 LLaMA      |  13B | 20.1   | 23.4   | 28.1   | 31.9
 LLaMA      |  33B |**24.9**| 28.3   | 32.9   | 36.0
 LLaMA      |  65B | 23.8   |**31.0**|**35.0**|**39.9**

*表4：__NaturalQuestions.__ Exact match performance.*


 Model      | psz  | 0-shot | 1-shot | 5-shot | 64-shot
------------|------|--------|--------|--------|----------
 Gopher     | 280B | 43.5   | -      | 57.0   | 57.2
 Chinchilla | 70B  | 55.4   | -      | 64.1   | 64.6
 LLaMA      |  7B  | 50.0   | 53.4   | 56.3   | 57.6
 LLaMA      | 13B  | 56.6   | 60.5   | 63.1   | 64.0
 LLaMA      | 33B  | 65.1   | 67.9   | 69.9   | 70.4
 LLaMA      | 65B  |**68.2**|**71.6**|**72.6**|**73.0**

*表 5：TriviaQA. Zero-shot and few-shot exact match performance on the filtered dev set.*

在这两个基准测试中，LLaMA-65B 在零样本和少样本设置中都实现了 state-of-the-arts 的性能。 更重要的是，LLaMA-13B 在这些基准测试中与 GPT-3 和 Chinchilla 相比也具有竞争力，尽管参数只有后者的 10%~20％（5-10 smaller）。 在推理场景，LLaMA-13B 能在单个 V100 GPU 上运行。

### 3.3 阅读理解（Reading Comprehension）

阅读理解能力测试基于 “RACE 阅读理解基准测试”（Lai 等，2017）。 这个数据集是从为中国初中生和高中生设计的英文阅读理解考试中收集的。 一些设置遵循 Brown 等（2020），测试结果见表 6，

 Model | psz  | RACE-middle | RACE-high
-------|------|-------------|-----------
 GPT-3 | 175B | 58.4        | 45.5
 PaLM  |   8B | 57.9        | 42.3
 PaLM  |  62B | 64.3        | 47.5
 PaLM  | 540B | **68.1**    | 49.1
 LLaMA |   7B | 61.1        | 46.9
 LLaMA |  13B | 61.6        | 47.2
 LLaMA |  33B | 64.1        | 48.3
 LLaMA |  65B | 67.9        | **51.6**

---
layout: post
title: 【论文】LLaMA：开放和高效的基础语言模型集
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

#### 2.1.2 Tokenizer（标记符）

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

 Model      | Params | BoolQ | PIQA | SIQA | HellaSwag | WinoGrande | ARC-e | ARC-c | OBQA
------------|--------|-------|------|------|-----------|------------|-------|-------|---------------
 GPT-3      | 175B   | 60.5  | 81.0 | -    | 78.9      | 70.2       | 68.8  | 51.4  | 57.6
 Gopher     | 280B   | 79.3  | 81.8 | 50.6 | 79.2      | 70.1       | -     | -     | -
 Chinchilla | 70B    | 83.7  | 81.8 | 51.3 | 80.8      | 74.9       | -     | -     | -
 PaLM       | 62B    | 84.8  | 80.5 | -    | 79.7      | 77.0       | 75.2  | 52.5  | 50.4
 PaLM-cont  | 62B    | 83.9  | 81.4 | -    | 80.6      | 77.0       | -     | -     | -
 PaLMa      | 540B   |**88.0**|82.3 | -    | 83.4      |**81.1**    | 76.6  | 53.0  | 53.4
 LLaMA      | 7B     | 76.5  | 79.8 | 48.9 | 76.1      | 70.1       | 72.8  | 47.6  | 57.2
 LLaMA      | 13B    | 78.1  | 80.1 | 50.4 | 79.2      | 73.0       | 74.8  | 52.7  | 56.4
 LLaMA      | 33B    | 83.1  | 82.3 | 50.4 | 82.8      | 76.0       |**80.0**|**57.8**| 58.6
 LLaMA      | 65B    | 85.3  |**82.8**|**52.3**|**84.2**|77.0       | 78.9  | 56.0  |**60.2**

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

 Model      | Params | 0-shot | 1-shot | 5-shot | 64-shot
------------|--------|--------|--------|--------|----------
 GPT-3      | 175B   | 14.6   | 23.0   |    -   | 29.9
 Gopher     | 280B   | 10.1   |    -   | 24.5   | 28.2
 Chinchilla |  70B   | 16.6   |    -   | 31.5   | 35.5
 PaLM       |   8B   |  8.4   | 10.6   |    -   | 14.6
 PaLM       |  62B   | 18.1   | 26.5   |    -   | 27.6
 PaLM       | 540B   | 21.2   | 29.3   |    -   | 39.6
 LLaMA      |   7B   | 16.8   | 18.7   | 22.0   | 26.1
 LLaMA      |  13B   | 20.1   | 23.4   | 28.1   | 31.9
 LLaMA      |  33B   |**24.9**| 28.3   | 32.9   | 36.0
 LLaMA      |  65B   | 23.8   |**31.0**|**35.0**|**39.9**

*表4：__NaturalQuestions.__ Exact match performance.*


 Model      | Params | 0-shot | 1-shot | 5-shot | 64-shot
------------|--------|--------|--------|--------|----------
 Gopher     | 280B   | 43.5   | -      | 57.0   | 57.2
 Chinchilla | 70B    | 55.4   | -      | 64.1   | 64.6
 LLaMA      |  7B    | 50.0   | 53.4   | 56.3   | 57.6
 LLaMA      | 13B    | 56.6   | 60.5   | 63.1   | 64.0
 LLaMA      | 33B    | 65.1   | 67.9   | 69.9   | 70.4
 LLaMA      | 65B    |**68.2**|**71.6**|**72.6**|**73.0**

*表 5：TriviaQA. Zero-shot and few-shot exact match performance on the filtered dev set.*

在这两个基准测试中，LLaMA-65B 在零样本和少样本设置中都实现了 state-of-the-arts 的性能。 更重要的是，LLaMA-13B 在这些基准测试中与 GPT-3 和 Chinchilla 相比也具有竞争力，尽管参数只有后者的 10%~20％（5-10 smaller）。 在推理场景，LLaMA-13B 能在单个 V100 GPU 上运行。

### 3.3 阅读理解（Reading Comprehension）

阅读理解能力测试基于 “RACE 阅读理解基准测试”（Lai 等，2017）。 这个数据集是从为中国初中生和高中生设计的英文阅读理解考试中收集的。 一些设置遵循 Brown 等（2020），测试结果见表 6，

 Model | Params | RACE-middle | RACE-high
-------|--------|-------------|-----------
 GPT-3 | 175B   | 58.4        | 45.5
 PaLM  |   8B   | 57.9        | 42.3
 PaLM  |  62B   | 64.3        | 47.5
 PaLM  | 540B   | **68.1**    | 49.1
 LLaMA |   7B   | 61.1        | 46.9
 LLaMA |  13B   | 61.6        | 47.2
 LLaMA |  33B   | 64.1        | 48.3
 LLaMA |  65B   | 67.9        | **51.6**

*表6：__阅读理解。__Zero-shot accuracy.*

### 3.4 数学推理（Mathematical reasoning）

在两个数学推理基准测试上评估模型：

- MATH（Hendrycks 等，2021）：一个包含 12K 个初中和高中数学问题的数据集，LaTeX 格式；
- GSM8k（Cobbe 等，2021）：一个初中数学问题集。

表 7 比较了 PaLM 和 Minerva（Lewkowycz 等，2022）。

 Model    | Params | MATH | +maj1@k | GSM8k | +maj1@k
----------|--------|------|---------|-------|-------
 PaLM     |   8B   | 1.5  | -       |  4.1  | -
 PaLM     |  62B   | 4.4  | -       | 33.0  | -
 PaLM     | 540B   | 8.8  | -       | 56.5  | -
 Minerva  |   8B   | 14.1 | 25.4    | 16.2  | 28.4
 Minerva  |  62B   | 27.6 | 43.4    | 52.4  | 68.5
 Minerva  | 540B|  **33.6**|**50.3**|**68.5**|**78.5**
 LLaMA    |   7B   | 2.9  |  6.9    | 11.0  | 18.1
 LLaMA    |  13B   | 3.9  |  8.8    | 17.8  | 29.3
 LLaMA    |  33B   | 7.1  | 15.2    | 35.6  | 53.1
 LLaMA    |  65B   | 10.6 | 20.5    | 50.9  | 69.7

*表 7：量化推理数据集（quantitative reasoning datasets）上的模型性能。For majority voting, we use the same setup as Minerva, with k = 256 samples for MATH and k = 100 for GSM8k (Minerva 540B uses k = 64 for MATH and and k = 40 for GSM8k). LLaMA-65B outperforms Minerva 62B on GSM8k, although it has not been fine-tuned on mathematical data.*

- Minerva 是一系列在 ArXiv 和 Math Web Pages 中提取的 38.5B token 上 finetune 而成的 PaLM 模型，
- PaLM 和 LLaMA 都没有在数学数据上进行 finetune 。

PaLM 和 Minerva 的性能数字取自 Lewkowycz 等（2022），我们分别用和不用 maj1@k 进行了比较。 maj1@k 表示我们为每个问题生成 k 个样本，并进行多数投票（Wang 等，2022）。

在 GSM8k 上，可以看到 LLaMA-65B 优于 Minerva-62B，尽管它没有在数学数据上进行微调。

### 3.5 代码生成（Code generation）

评估模型从给出的自然语言描述来生成代码的能力，使用了两个基准测试：

1. HumanEval（Chen 等，2021）
2. MBPP（Austin 等，2021）

这两个测试，都是给模型几句关于程序的描述，以及一些输入输出示例。

> In HumanEval, it also receives a function signature, and the prompt is formatted as natural code with the textual description and tests in a program that fits the description and satisfies the test cases.

在表 8 中，我们将 LLaMA 的 pass@1 得分与未在代码上进行微调的现有语言模型进行了比较，即 PaLM 和 LaMDA（Thoppilan 等，2022）。 PaLM 和 LLaMA 是在包含相似数量的代码 token 的数据集上训练的。


 -         | Params | HumanEval | HumanEval | MBPP | MBPP
-----------|--------|-----------|-----------|------|------
 pass@     | -      | @1        | @100      | @1   | @80
 LaMDA     | 137B   | 14.0      | 47.3      | 14.8 | 62.4
 PaLM      | 8B     | 3.6∗      | 18.7∗     | 5.0∗ | 35.7∗
 PaLM      | 62B    | 15.9      | 46.3∗     | 21.4 | 63.2∗
 PaLM-cont | 62B    | 23.7      | -         | 31.2 | -
 PaLM      | 540B   |**26.2**   | 76.2      | 36.8 | 75.0
 LLaMA     | 7B     | 10.5      | 36.5      | 17.7 | 56.2
 LLaMA     | 13B    | 15.8      | 52.5      | 22.0 | 64.0
 LLaMA     | 33B    | 21.7      | 70.7      | 30.2 | 73.4
 LLaMA     | 65B    | 23.7      |**79.3**   |**37.7**|**76.8**

*表 8：Model performance for code generation. We report the pass@ score on HumanEval and MBPP. HumanEval generations are done in zero-shot and MBBP with 3-shot prompts similar to Austin et al. (2021). The values marked with are read from figures in Chowdhery et al. (2022).*

如表 8 所示，

- 对于类似数量的参数，LLaMA 优于其他一般模型，如 LaMDA 和 PaLM，它们没有专门针对代码进行训练或微调。
- LLaMA 具有 13B 参数及以上，在 HumanEval 和 MBPP 上均优于 LaMDA 137B。
- LLaMA 65B 也优于 PaLM 62B，即使它的训练时间更长。

> 本表中 pass@1 结果是通过 temperature=0.1 采样得到的。 pass@100 和 pass@80 指标是通过 temperature=0.8 获得的。 我们使用与 Chen 等（2021）相同的方法来获得 pass@k 的无偏估计。

通过在代码特定 token 上进行微调，可以提高生成代码的性能。例如，

- PaLM-Coder（Chowdhery 等，2022）将 PaLM 在 HumanEval 上的 pass@1 分数从 PaLM 的 26.2％提高到 36％。
- 其他专门针对代码进行训练的模型在这些任务上也表现比通用模型更好（Chen 等，2021; Nijkamp 等，2022; Fried 等，2022）。

在代码 token 上进行微调超出了本文的范围。

### 3.6 大规模多任务语言理解（Massive Multitask Language Understanding）

大规模多任务语言理解基准测试（MMLU）由 Hendrycks 等（2020）提出， 包括涵盖人文、STEM 和社会科学等各种知识领域的多项选择题。 我们在 5-shot 设置下使用基准测试提供的示例来评估我们的模型，结果如表 9 所示，

 Models     | Params | Humanities | STEM | Social Sciences | Other | Average
------------|--------|------------|------|-----------------|-------|-----------
 GPT-NeoX   |  20B   | 29.8       | 34.9 | 33.7            | 37.7  | 33.6
 GPT-3      | 175B   | 40.8       | 36.7 | 50.4            | 48.8  | 43.9
 Gopher     | 280B   | 56.2       | 47.4 | 71.9            | 66.1  | 60.0
 Chinchilla |  70B   | 63.6       | 54.9 | 79.3           |**73.9**| 67.5
 PaLM       |   8B   | 25.6       | 23.8 | 24.1            | 27.8  | 25.4
 PaLM       |  62B   | 59.5       | 41.9 | 62.7            | 55.8  | 53.7
 PaLM       | 540B   |**77.0**   |**55.6**|**81.0**        | 69.6  |**69.3**
 LLaMA      |   7B   | 34.0       | 30.5 | 38.3            | 38.1  | 35.1
 LLaMA      |  13B   | 45.0       | 35.8 | 53.8            | 53.3  | 46.9
 LLaMA      |  33B   | 55.8       | 46.0 | 66.7            | 63.4  | 57.8
 LLaMA      |  65B   | 61.8       | 51.7 | 72.9            | 67.4  | 63.4

*表 9：Massive Multitask Language Understanding (MMLU). Five-shot accuracy*

可以看到，LLaMA-65B 落后于 Chinchilla-70B 和 PaLM-540B 几个百分点，并且在大部分领域都是如此。 一个可能的解释是我们在预训练数据中使用了有限数量的书籍和学术论文，即 ArXiv、Gutenberg 和 Books3，总共只有 **177GB**， 而后两个模型是在多达 **2TB** 的书籍上进行训练的。 Gopher、Chinchilla 和 PaLM 使用的大量书籍可能也解释了为什么 Gopher 在这个基准测试中表现优于 GPT-3，而在其他基准测试中表现只是差不多。


### 3.7 训练过程中性能的变化

在训练过程中，我们跟踪了 LLaMA 在一些问题回答和常识基准测试上的性能，如图 2，

![figure 2:](/postimgs/llama1_figure2_evolution_of_performance_on_qa_and_com_sense_reasoning.jpg)

*图 2：Evolution of performance on question answering and common sense reasoning during training*

在大多数基准测试中，性能随着 token 数量稳步提高，并与模型的 training perplexity 相关（见图 1）。

![figure 1: Training loss over train tokens for the 7B, 13B, and 65B models](/postimgs/llama1_training_loss.jpg)

*图1：7B, 13B, 33B, 和 65B 模型的训练损失。LLaMA-33B 和 LLaMA-65B 是在 1.4T token 上训练的，小点的模型是在 1.0T token 上训练的，所有模型的训练 batch 大小为 4M token*

## 4. 指令微调（Instruction Finetuning）

在本节中，我们将说明简单地在指令数据上进行微调，就会迅速提高在 MMLU 上的性能。

尽管 LLaMA-65B 的未微调版本已经能够 follow 基本指令，但我们观察到进行一点微调可以提高在 MMLU 上的性能， 并能进一步提高模型 follow 指令的能力。 由于这不是本文的重点，我们只进行了一次实验，遵循 Chung 等（2022）的相同协议来训练一个指令模型 LLaMA-I。 LLaMA-I 在 MMLU 上的结果见表 10，与当前中等规模的指令微调模型 OPT-IML（Iyer 等，2022）和 Flan-PaLM 系列（Chung 等，2022）进行了比较：

 Model          | Params | MMLU
----------------|--------|-----------
 OPT            | 30B    | 26.1
 GLM            | 120B   | 44.8
 PaLM           | 62B    | 55.1
 PaLM-cont      | 62B    | 62.8
 Chinchilla     | 70B    | 67.5
 LLaMA          | 65B    | 63.4
 OPT-IML-Max    | 30B    | 43.2
 Flan-T5-XXL    | 11B    | 55.1
 Flan-PaLM      | 62B    | 59.6
 Flan-PaLM-cont | 62B    | 66.1
 LLaMA-I        | 65B    |**68.9**

*表 10：Instruction finetuning – MMLU (5-shot). Comparison of models of moderate size with and without instruction finetuning on MMLU.*

尽管这里使用的指令微调方法很简单，但我们在 MMLU 上达到了 68.9％。 LLaMA-I（65B）在 MMLU 上优于现有的中等规模指令微调模型，但仍远远落后于最先进的 GPT code-davinci-002 在 MMLU 上的 77.4（数字来自 Iyer 等（2022））。有关 57 个任务的 MMLU 性能详细信息，请参见附录的表 16。

## 5. 偏见、毒性和错误信息

大型语言模型已被证明会重现和放大训练数据中存在的偏差（Sheng 等人，2019 年；Kurita 等人，2019 年），并生成有毒或令人反感的内容（Gehman 等人，2020 年）。 由于我们的训练数据集包含很大一部分来自网络的数据，因此我们认为确定我们的模型生成此类内容的潜力至关重要。 为了了解 LLaMA-65B 的潜在危害，我们对测量有毒成分产生和刻板印象检测的不同基准进行了评估。 虽然我们选择了语言模型社区使用的一些标准基准来指示这些模型的一些问题，但这些评估不足以完全理解与这些模型相关的风险。


### 5.1 真实毒性提示

语言模型可能会生成有毒语言，例如侮辱、仇恨言论或威胁。 模型可以生成的有毒内容范围非常大，这使得彻底的评估具有挑战性。 最近的几项工作（Zhang 等人，2022；Hoffmann 等人，2022）已将 RealToxicityPrompts 基准（Gehman 等人，2020）视为其模型毒性程度的指标。 RealToxicityPrompts 包含模型必须完成的大约 100k 个提示； 然后通过向 PerspectiveAPI 3 发出请求来自动评估毒性分数。

我们无法控制第三方 PerspectiveAPI 使用的管道，因此很难与以前的模型进行比较。 对于 100k 提示中的每一个，我们都会用我们的模型贪婪地生成，并测量它们的毒性分数。 每个提示的分数范围从 0（无毒）到 1（有毒）。 在表 11 中，我们报告了 RealToxicityPrompts 的基本提示类别和尊重提示类别的平均得分。 这些分数与我们在文献中观察到的分数“相当”（例如，Chinchilla 为 0.087），但这些工作与我们的方法之间存在差异（在采样策略、提示数量和 API 时间方面）。 我们观察到毒性随着模型的大小而增加，特别是对于尊重提示。 在之前的工作中也观察到了这一点（Zhang 等人，2022），但 Hoffmann 等人的显着例外。 （2022）尽管尺寸不同，但他们没有看到龙猫和地鼠之间的差异。 这可以通过以下事实来解释：较大的模型 Gopher 的性能比 Chinchilla 差，这表明毒性和模型大小之间的关系可能仅适用于模型家族。

 Model  | Params | Basic | Respectful
--------|--------|-------|-------------
 LLaMA  |  7B    | 0.106 | 0.081
 LLaMA  | 13B    | 0.104 | 0.095
 LLaMA  | 33B    | 0.107 | 0.087
 LLaMA  | 65B    | 0.128 | 0.141

*表11: RealToxicityPrompts. We run a greedy decoder on the 100k prompts from this benchmark. The “respectful” versions are prompts starting with “Complete the following sentence in a polite, respectful, and unbiased manner:”, and “Basic” is without it. Scores were obtained using the PerplexityAPI, with higher score indicating more toxic generations.*

### 5.2 CrowS-Pairs

我们评估了 CrowSPairs 模型中的偏差（Nangia 等人，2020）。 该数据集可以衡量 9 个类别的偏见：性别、宗教、种族/肤色、性取向、年龄、国籍、残疾、外貌和社会经济地位。 每个例子都由刻板印象和反刻板印象组成，我们使用零样本设置中两个句子的困惑度来测量刻板句子的模型偏好。 因此，分数越高表明偏差越大。 我们在表 12 中与 GPT-3 和 OPT-175B 进行比较。

平均而言，LLaMA 比这两种模型略胜一筹。 我们的模型在宗教类别中尤其存在偏差（与 OPT-175B 相比+10%），其次是年龄和性别。 尽管有多个过滤步骤，但我们预计这些偏差仍来自 CommonCrawl。

 -                   | LLaMA | GPT3 | OPT
---------------------|-------|------|------
 Gender              | 70.6  | 62.6 | 65.7
 Religion            | 79.0  | 73.3 | 68.6
 Race/Color          | 57.0  | 64.7 | 68.6
 Sexual orientation  | 81.0  | 76.2 | 78.6
 Age                 | 70.1  | 64.4 | 67.8
 Nationality         | 64.2  | 61.6 | 62.9
 Disability          | 66.7  | 76.7 | 76.7
 Physical appearance | 77.8  | 74.6 | 76.2
 Socioeconomic status| 71.5  | 73.8 | 76.2
 Average             | 66.6  | 67.2 | 69.5

*表12：CrowS-Pairs. We compare the level of biases contained in LLaMA-65B with OPT-175B and GPT3-175B. Higher score indicates higher bias.*

### 5.3 WinoGender

为了进一步研究我们的模型在性别类别上的偏差，我们查看了 WinoGender 基准（Rudinger 等人，2018），这是一个共同参考分辨率数据集。 WinoGender 由 Winograd 模式组成，通过确定模型共指解析性能是否受到代词性别的影响来评估偏差。

更准确地说，每个句子都有三个提及：“职业”、“参与者”和“代词”，其中代词共同指代职业或参与者。 我们提示模型确定共指关系，并根据句子的上下文来衡量它是否正确。 目标是揭示模型是否捕获了与职业相关的社会偏见。 例如，WinoGender 数据集中的一句话是“护士通知病人他的轮班将在一小时后结束。”，后面跟着“His”所指的。 然后，我们比较护士和患者连续的困惑度，以与模型进行共参考解决。 我们评估使用 3 个代词时的表现：“她/她/她”、“他/他/他”和“他们/他们/某人”（与代词的语法功能相对应的不同选择。

在表 13 中，我们报告了数据集中包含的三个不同代词的共指分数。 我们观察到，我们的模型在执行“他们/他们/某人”代词的共指解析方面明显优于“她/她/她”和“他/他/他”代词。 之前的工作中也进行了类似的观察（Rae 等人，2021；Hoffmann 等人，2022），这可能表明存在性别偏见。 事实上，在“她/她/她”和“他/他/他”代词的情况下，模型可能使用职业的大多数性别来执行共指解析，而不是使用句子的证据 。

为了进一步研究这一假设，我们查看了 WinoGender 数据集中“her/her/she”和“his/him/he”代词的“陷阱”案例集。 这些情况对应的句子中代词与职业的大多数性别不匹配，而职业是正确答案。 在表 13 中，我们观察到我们的模型 LLaMA-65B 在陷阱示例上犯了更多错误，清楚地表明它捕获了与性别和职业相关的社会偏见。 “她/她/她”和“他/他/他”代词的表现有所下降，这表明无论性别如何，都存在偏见。

 -                    | 7B   | 13B  | 33B  | 65B
----------------------|------|------|------|------
 All                  | 66.0 | 64.7 | 69.0 | 77.5
 her/her/she          | 65.0 | 66.7 | 66.7 | 78.8
 his/him/he           | 60.8 | 62.5 | 62.1 | 72.1
 their/them/someone   | 72.1 | 65.0 | 78.3 | 81.7
 her/her/she (gotcha) | 64.2 | 65.8 | 61.7 | 75.0
 his/him/he (gotcha)  | 55.0 | 55.8 | 55.8 | 63.3

*表13：WinoGender. Co-reference resolution accuracy for the LLaMA models, for different pronouns (“her/her/she” and “his/him/he”). We observe that our models obtain better performance on “their/them/someone’ pronouns than on “her/her/she” and “his/him/he’, which is likely indicative of biases.*


 Model | Params | Truthful | Truthful*Inf
-------|--------|----------|--------------
 GPT-3 | 1.3B   | 0.31     | 0.19
 GPT-3 |   6B   | 0.22     | 0.19
 GPT-3 | 175B   | 0.28     | 0.25
 LLaMA |   7B   | 0.33     | 0.29
 LLaMA |  13B   | 0.47     | 0.41
 LLaMA |  33B   | 0.52     | 0.48
 LLaMA |  65B   | 0.57     | 0.53

_表14：TruthfulQA. We report the fraction of truthful and truthful*informative answers, as scored by specially trained models via the OpenAI API. We follow the QA prompt style used in Ouyang et al. (2022), and report the performance of GPT-3 from the same paper._

在表 14 中，我们报告了我们的模型在两个问题上的表现，以衡量真实模型以及真实与信息的交集。 与 GPT-3 相比，我们的模型在两个类别上的得分都较高，但正确答案率仍然较低，这表明我们的模型很可能会产生错误答案。

## 6.碳足迹（Carbon footprint）

训练 LLaMA 消耗了大量能源，排放了很多二氧化碳。我们遵循最近的文献，将总能耗和产生的碳足迹分解在表 15 中，

 Model      | GPU Type  | GPU Power |  GPU-hours | Total power |  Carbon emitted
------------|-----------|-----------|------------|-------------|-----------------
 OPT-175B   | A100-80GB | 400W      | 809,472    | 356 MWh     | 137
 BLOOM-175B | A100-80GB | 400W      | 1,082,880  | 475 MWh     | 183
 LLaMA-7B   | A100-80GB | 400W      | 82,432     |  36 MWh     | 14
 LLaMA-13B  | A100-80GB | 400W      | 135,168    |  59 MWh     | 23
 LLaMA-33B  | A100-80GB | 400W      | 530,432    | 233 MWh     | 90
 LLaMA-65B  | A100-80GB | 400W      | 1,022,362  | 449 MWh     | 173

*表 15： Carbon footprint of training different models in the same data center. We follow Wu et al. (2022) to compute carbon emission of training OPT, BLOOM and our models in the same data center. For the power consumption of a A100-80GB, we take the thermal design power for NVLink systems, that is 400W. We take a PUE of 1.1 and a carbon intensity factor set at the national US average of 0.385 kg CO2e per KWh.*

我们采用 Wu 等(2022)的公式来估算训练模型所需的瓦时数（Watt-hour, Wh）和碳排放量（carbon emissions）。 对于瓦时数，我们使用以下公式：

Wh = GPU-h * (GPU power consumption) * PUE

其中，我们的功率使用效率（PUE）为 1.1。 产生的碳排放量取决于用于训练所在的数据中心的位置。例如，

- BLOOM 使用排放 0.057kg CO2eq/KWh 的电网，产生 27 tCO2eq 的排放量，
- OPT 使用排放 0.231kg CO2eq/KWh 的电网，导致 82 tCO2eq 的排放量。

在本研究中，我们感兴趣的是在同一个数据中心的情况下，不同模型训练的碳排放成本。 因此，我们不考虑数据中心的位置，并使用美国国家平均碳强度系数（carbon intensity factor） 0.385kg CO2eq/KWh。 那么此时就有，

tCO2eq = MWh * 0:385

我们对 OPT 和 BLOOM 采用相同的公式进行公平比较。

- 对于 OPT，我们假设训练需要在 992 个 A100-80GB 上进行 34 天（参见他们的日志 4）。
- 我们在 2048 个 A100 80GB 上，用了约 5 个月时间来开发 LLaMA。 根据前面的公式，计算得到 LLaMA 的训练成本约为 2638 MWh，总排放量为 1015 tCO2eq。

我们希望 LLaMA 的发布有助于减少未来的碳排放，因为它训练已经完成（很多情况下大家直接用或者进行微调就行了）: 而且其中一些小参数模型可以在单个 GPU 上运行。

## 7.相关工作（Related work）

语言模型是单词、 token 或字符组成的序列的概率分布 （probability distributions over sequences of words, tokens or characters）(Shannon, 1948, 1951)。

这个任务通常被描述为对下一个 token 的预测，在自然语言处理（Bahl 等，1983；Brown 等，1990）中很早就是一个核心问题了。 Turing（1950）提出通过“模仿游戏”（imitation game），使用语言来衡量机器智能， 因此语言建模（language modeling）成为了衡量人工智能进展的基准（Mahoney，1999）。

### 7.1 架构

传统上，语言模型基于 n-gram 计数统计（Bahl 等人，1983），并且提出了各种平滑技术来改进罕见事件的估计（Katz，1987；Kneser 和 Ney，1995）。 在过去的二十年里，神经网络已经成功地应用于语言建模任务，从前馈模型（Bengio et al., 2000）、循环神经网络（Elman, 1990; Mikolov et al., 2010）和LSTM（ Hochreiter 和 Schmidhuber，1997；Graves，2013）。 最近，基于自注意力的 Transformer 网络取得了重要的改进，特别是在捕获长距离依赖性方面（Vaswani 等人，2017 年；Radford 等人，2018 年；Dai 等人，2019 年）。

### 7.2 Scaling

语言模型的缩放（模型和数据集大小）有着悠久的历史。 布兰茨等人。 (2007) 展示了使用在 2 万亿个标记上训练的语言模型（产生 3000 亿个 n 元语法）对机器翻译质量的好处。 虽然这项工作依赖于一种简单的平滑技术，称为 Stupid Backoff，Heafield 等人。 (2013) 后来展示了如何将 Kneser-Ney 平滑扩展到网络规模的数据。 这允许在 CommonCrawl 的 9750 亿个令牌上训练 5-gram 模型，从而产生一个具有 5000 亿个 n-gram 的模型（Buck 等人，2014 年）。 切尔巴等人。 (2013) 引入了十亿字基准，这是一个用于衡量语言模型进展的大规模训练数据集。

在神经语言模型的背景下，Jozefowicz 等人。 (2016) 通过将 LSTM 扩展到 10 亿个参数，在 Billion Word 基准上获得了最先进的结果。 后来，缩放转换器导致许多 NLP 任务的改进。 著名的模型包括 BERT（Devlin 等人，2018）、GPT-2（Radford 等人，2019）、Megatron-LM（Shoeybi 等人，2019）和 T5（Raffel 等人，2020）。 GPT-3（Brown 等人，2020）取得了重大突破，该模型拥有 1750 亿个参数。 这催生了一系列大型语言模型，例如 Jurassic-1 (Lieber et al., 2021)、Megatron-Turing NLG (Smith et al., 2022)、Gopher (Rae et al., 2021)、Chinchilla (Hoffmann) 等人，2022）、PaLM（Chowdhery 等人，2022）、OPT（Zhang 等人，2022）和 GLM（Zeng 等人，2022）。 赫斯特内斯等人。 （2017）和罗森菲尔德等人。 (2019) 研究了缩放对深度学习模型性能的影响，表明模型和数据集大小与系统性能之间存在幂律。 卡普兰等人。 （2020）专门针对基于 Transformer 的语言模型推导了幂律，后来由 Hoffmann 等人进行了完善。 （2022），通过在缩放数据集时调整学习率计划。 最后，魏等人。 (2022) 研究了缩放对大型语言模型能力的影响。

## 8. 结论

在本文中，我们提出了一系列公开发布的语言模型，并与最先进的基础模型竞争。 最值得注意的是，LLaMA-13B 的性能优于 GPT-3，同时尺寸缩小了 10 倍以上，LLaMA-65B 与 Chinchilla-70B 和 PaLM-540B 具有竞争力。 与之前的研究不同，我们表明，通过专门针对公开可用的数据进行训练，无需求助于专有数据集，就可以实现最先进的性能。 我们希望向研究界发布这些模型将加速大型语言模型的开发，并有助于提高其稳健性并减轻诸如毒性和偏见等已知问题。 此外，我们像 Chung 等人一样观察到。 （2022）根据指令微调这些模型会带来有希望的结果，我们计划在未来的工作中进一步研究这一点。 最后，我们计划将来发布在更大的预训练语料库上训练的更大的模型，因为我们在扩展过程中看到了性能的不断提高。

> https://arxiv.org/pdf/2302.13971.pdf
> http://arthurchiao.art/blog/llama-paper-zh/

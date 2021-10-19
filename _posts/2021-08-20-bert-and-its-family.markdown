---
layout: post
title: Bert 及其家族简介
category: Document
tags: dl bert
date: 2021-08-20 10:08:21
published: true
summary: 介绍预训练向量模型和微调
image: pirates.svg
comment: true
---

*芝麻街任务命名*

- ELMo: Embeddings from Language Models
- BERT: Bidirectional Encoder Representations from Transformers
- ERNIE: Enhanced Representation through kNowledge IntEgration
- Grover: Generating aRticles by Only Viewing mEtadata Records

## 预训练模型

- 把每个词表示为一个嵌入向量，同样的词有相同的嵌入（Represent each token by a embedding vector）
    - Word2vec
    - Glove
    - FastText
- Contextualized Word Embedding

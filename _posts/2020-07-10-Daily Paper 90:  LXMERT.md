---
layout: post
title: "Daily Paper 90: LXMERT: Learning Cross-Modality Encoder Representations from Transformers"
description: "Notes"
categories: [MMML-Transformer]
tags: [Paper]
redirect_from:
  - /2020/07/10/
---

# Daily Paper 90: LXMERT: Learning Cross-Modality Encoder Representations from Transformers  

## Introduction  

这篇文章是昨天看上一篇的时候看到的，也是一个多模态的transformer，由北卡的Hao Tan和Mohit Bansal发表在EMNLP2019上。LXMERT全称为Learning Cross-Modality Encoder Representations from Transformers，也就是用Transformer来学习跨模态encoder表示。具体来讲，作者建立了一个大的transformer，该模型主要由三部分组成：一个object relationship encoder，一个language encoder和一个跨模态的encoder，之后在五个预训练任务上进行了预训练，分别是masked language modeling, masked object prediction(特征回归和标签分类), cross-modality matching和image question answering。这些任务能够学习到跨模态和单模态的信息。经过预训练之后再进行fine-tune，从而在两个VQA数据集上得到了SOTA的表现。此外，作者发现他们的预训练跨模态模型可以泛化到



---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

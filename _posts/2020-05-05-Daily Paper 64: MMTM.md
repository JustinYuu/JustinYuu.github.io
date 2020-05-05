---
layout: post
title: "Daily Paper 64: MMTM"
description: "Notes"
categories: [MMML-Fusion]
tags: [Paper]
redirect_from:
  - /2020/05/05/
---

# Daily Paper 64: MMTM: Multimodal Transfer Module for CNN Fusion  

## Introduction  

这篇文章是微软和Georgia Tech在三月底挂在arxiv上的，提出了一个Multimodal Transfer Module的intermediate fusion模型，用于CNN的特征融合。MMTM利用了多个模态之间的信息来对每一个CNN流的channel-wise特征进行重新校准。与其他intermediate fusion方法不同的是，MMTM能够在不同的空间维度卷积层间进行特征融合，并能够在仅改变很小的单模态网络架构前提下进行多模态特征融合。作者在动态手势识别、演讲增强、动作识别任务上都取得了很好的效果。  

为什么大家都使用late fusion呢，作者认为是因为late fusion比较简单粗暴，很好操作，比如如果使用注意力或者池化来fuse每个维度的1维预测值。而intermediate fusion由于需要在中间环节进行维度的匹配和对齐，因此会导致整个过程比较复杂。此外由于late fusion的每一个stream的操作都经过详细的单模态研究，因此可以直接把这些单模态的研究成果搬来用，或者直接使用CNN的预训练权重，而intermediate fusion需要对底层的框架进行彻底的修改，从而导致预训练权重和之前的单模态处理方式都无法直接拿来用，所以对这部分的研究还比较少。  

作者对intermediate fusion的研究就是为了解决上述的弊端，作者的灵感来源于SENet的squeeze and excitation模块，







---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

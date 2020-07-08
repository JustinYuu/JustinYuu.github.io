---
layout: post
title: "Daily Paper 89: Adaptive Transformers"
description: "Notes"
categories: [NLP]
tags: [Paper]
redirect_from:
  - /2020/07/08/
---

# Daily Paper 89: Adaptive Transformers for Learning Multimodal Representations  

## Introduction  

这篇文章发表在ACL2020 Student Research Workshop上，主要是对Transformer的改进。作者认为之前的transformer虽然有很广泛的应用，但是参数太多，需要巨大的计算量。因此作者对transformer进行了扩展，使用一些适应性方法来获得更多的模型可解释性和计算效率。具体来，作者研究了注意力的分散，稀疏和结构化的dropout方法，以帮助了解他们的注意力机制如何扩展到视觉和语言任务，此外作者还证明了这些方法能够使得其更了解网络是如何处理复杂的输入序列和不同模态之间的稀疏偏好的。  

总体来讲，这篇文章的贡献如下：首个即将自适应方法从语言特征扩展到多模态特征上，以研究它们如何对齐以捕获不同模态之间的复杂关系，并研究了对齐这些方法带来的影响，从而通过消融分析了解其兼容性；进行了可解释性分析，以了解这些方法如何增强人类对注意力行为和适应性方法的理解；提供了针对多模态输入序列的最新自适应方法的实验结果。总结一下就是搬了一个叫做自适应的方法过来，然后进行了可解释性分析，并得到了不错的实验结果。  

## Background  

由于使用的方法应该不是作者独创的，所以background这一环节其实就相当于之前文章的method部分了。这里主要对作者使用的backbone网络，自适应注意力跨度、自适应稀疏注意力两种自适应方法，以及一个trick LayerDrop进行了介绍，下面分别详细介绍这几种方法。  

### LXMERT  

LXMERT是一个多模态的transformer，作者使用该网络作为baseline架构。自适应方法能够和任意形式的基于transformer的自注意力机制结合，而LXMERT使用自注意力和跨模态注意力来联合表示输入的图片和文本。具体来讲，它将一个词语级别的句子和物体级别的图像embedding作为输入，encoder由9层的单模态语言encoder、5层的单模态视觉encoder和5层的跨模态encoder组成，从而联合表示特征。作者使用的网络在四个任务上预训练过，分别是Masked Cross Modality LM, Masked Object Prediction, Cross Modality LM, Mask Object Prediction, Cross Modality Matching和Image Question Answering，其中视觉特征的ROI特征使用Faster RCNN提取。  

### Adaptive Attention Span  

接下来介绍自适应方法，第一个方法叫做Adaptive Attention Span，动态的attention span能够使得学习最佳的注意力span这一过程本身可以根据注意力头部确定的上下文大小来收集信息。首先介绍一下究竟什么是attention span。所谓的attention span，就是一个注意力头能够联系到的上下文范围大小。如果所有的头都能联系到全文，那么这几个头除了权重不同之外，学习的逻辑其实并没有什么差别。

这里要手动规定一个每个头的span的上界，从而减少计算复杂度和内存开销。由于不同的头根据任务的不同侧重于不同的上下文，作者显式的证明了不同头的注意力span会根据任务的复杂性不同而有非常明显的差异性。作者使用和Sukhbaatar等人几乎相同的设置，公式如下:  
![89-1](/images/daily paper/89-1.png)  

其中z是模型的参数，使用kaiming normal初始化，m<sub>z</sub>与注意力权重耦合，超参数R帮助控制注意力分布的softness。




---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

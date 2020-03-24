---
layout: post
title: "Daily Paper 54: Few-Shot Video Classification via Temporal Alignment"
description: "Notes"
categories: [CV-Meta]
tags: [Paper]
redirect_from:
  - /2020/03/24/
---

# Daily Paper 54: Few-Shot Video Classification via Temporal Alignment  

## Introduction  

这篇是Stanford在去年6月份传到arxiv上的，不知道为什么还没发表，主要提出了一个Temporal Alignment Module时序对齐模块。TAM能够通过计算query video沿着对齐路径的每一帧的距离，来计算query video关于新类别代理的总距离值。作者还引入了continuous relaxation模块，使得模型可以使用端到端的模式来学习和直接优化小样本学习目标。作者在Kinetics和Something-Something-V2数据集上进行了实验，得到了不错的效果。  

作者的idea其实很简单，就是把时序作为距离计算的考量标准，而不是简单的将时序特征池化压缩，作者认为这种压缩会丢失很多特征信息(这是肯定的)，所以作者就设置了某种帧对齐方式，然后逐帧进行比对。我认为这个idea是想起来容易，做起来难的类型。首先，使用C3D+GRU的所谓的压缩时序信息的过程是为了能够将视频的信息融合到一个向量中，在度量模块中进行对比，那么GRU首先就保证了时序信息也有一些考量。其次如果强行逐帧对比，那么提取的特征就不再是一个向量，而是一个矩阵，后续的度量模块也要重新设置。最后，由于视频的长度不同，逐帧对比究竟要选取哪些帧，也是要解决的问题。我个人觉得要解决这三点，才能实现这一idea。  

作者认为他们主要的contribution有三点：第一，作者首先显式的强调了小样本视频分类中的非线性时序多样性问题；第二，作者提出了TAM模块，能够动态的对齐两个视频序列，同时保持了其他工作经常忽略的时序顺序；第三，作者使用了continuous relaxation模块，使得其模块能够完全可微，实现了STOA的效果。  

这篇文章很大程度借鉴了CMN网络，但是作者认为CMN最大的问题在于帧被完全打乱，不具有时序特征，因此性能可能有所损失。作者对于帧对齐的灵感来源于动态规划，虽然动态规划能够保证找到最优的对齐方法，但是这一操作是离散不可微的，因此就没法训练，所以作者使用了continuous relaxation方法，将离散的方式转化为可训练的方式，从而能够在整体上端到端训练。  

## Method  

整体的流程图如下图所示:  
![54-1](/images/daily paper/54-1.png)  
整个流程由两个部分组成，embedding module和relation module。  

### Embedding module  

embedding module不仅要完成提取特征的任务，还要保证提取的特征能够对齐。事实上，一个视频有若干个帧，但是很多帧的信息都是冗余的，所以作者提前进行了一个预筛选，使用Temporal Segment Network的稀疏采样模式，将视频序列分为T个片段，然后从每一个片段里提取一小个片段，也就是说视频的所有信息都能被捕获，但是离散采样可以保证数量的相同和帧冗余现象的缓解。  

预采样之后，对于每一个片段，都使用一个CNN网络f<sub>ψ</sub>来进行编码，假设输入序列S中有T个片段，那么最后的特征向量也有T个，那么整个特征就是T×D<sub>f</sub>。我之前也说过，这一步并不是难题，最关键和困难的是如何进行矩阵之间的比较，我觉得这需要比较强的数学知识。  

### Distance Measure with TAM  





---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

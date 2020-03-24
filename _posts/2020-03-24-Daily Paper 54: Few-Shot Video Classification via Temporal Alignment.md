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

作者定义了距离矩阵D，其计算公式如下图:  
![54-2](/images/daily paper/54-2.png)  

通过该距离能够计算出两个视频之间的任意一对帧的距离，不过作者也不需要计算出所有的距离，只需要计算对应帧的距离就好了。作者定义了一个bool矩阵W作为是否对齐的判别标志，作者的目标就是要找到最好的对齐W，即W<sup>/*</sup> = argmin(W, D(f<sub>ψ</sub>(S<sub>i</sub>),f<sub>ψ</sub>(S<sub>j</sub>)))，使得W和D的内积最小，其内积也就是视频间的距离。作者使用Dynamic Time Warping算法的变体来计算W<sup>/*</sup>，该算法使用了一个累计距离函数来求解，公式如下图:  
![54-3](/images/daily paper/54-3.png)  

作者还对DTW做了一些小小的修改，由于DTW的矩阵中W<sub>11</sub>和W<sub>TT</sub>始终为1，这就说明整个视频的开头和结尾部分都会考量在内。但是作者认为在视频中可能一开始和结尾的部分有可能是不重要的，所以就修改了这个限制条件。具体的方法是在开始和结束padding了两列，将T×T的矩阵扩充成了T×(T+2)，累积距离函数也就变成了下图的形式:  
![54-4](/images/daily paper/54-4.png)  

此外作者还讲解了传说中的continious relaxation究竟是个什么东西。在之前的公式中，最为严重的事情是γ相对于D来说是不可微的。所以作者采取了log-sum-exp的方式，引入了一个超参数λ来使公式变的可微:  
![54-5](/images/daily paper/54-5.png)  

整体的训练方式如下:  
![54-6](/images/daily paper/54-6.png)  

## Experiments  

作者使用了在ImageNet上预训练的ResNet50作为特征提取器对视频的帧进行逐帧提取，然后将帧级别的特征平均得到一个视频级别的特征，作为小样本算法的输入部分。作者的baseline使用了Matching Net、MAML和CMN。  

在进行元学习调优的时候，作者使用了ICLR2019 A Closer Look at Few-Shot Learning这篇的Baseline++方法作为strong baselines，其中有TSN++，CMN++, TRN++。  

作者在Kinetics和Something-Something V2数据集上进行了5-way 1/5-shot分类任务，结果显示均实现了STOA，在Kinetics上的1/5-shot准确率分别为73.0,85.8，在Something V2上为42.8,52.3。  

实验细节、消融实验和可视化结果略过了。  

## Conclusion  

作者提出了一个时序对齐模块TAM，用来显式的学习距离衡量方法和独立于非线性时序变化的表示。TAM能够动态的对齐两个视频序列，并在与此同时保持时序。此外，该模型是一个端到端的模型，使用continuous relaxation来保证整个模型可以直接进行优化。最后的效果也非常好，总之还是挺不错的一篇paper，可以用来当baseline。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

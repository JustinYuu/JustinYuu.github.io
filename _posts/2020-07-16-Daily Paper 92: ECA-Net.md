---
layout: post
title: "Daily Paper 92: ECA-Net"
description: "Notes"
categories: [CV-Attention]
tags: [Paper]
redirect_from:
  - /2020/07/16/
---

# Daily Paper 92: ECA-Net: Efficient Channel Attention for Deep Convolutional Neural Networks  

## Introduction  

这篇paper是天大、大工和哈工大共同发表的，提出了一个用于卷积网络的新的channel注意力机制。有关channel attention的文章最近可谓多如牛毛，这是因为channel attention能够有效的提升深度卷积神经网络的潜在性能，从而提升网络的表现。然而作者认为目前存在的种种方法都力图引入更为复杂的注意力模块来得到更好的表现，这不可避免的增加了更多的复杂度。因此为了克服复杂度和网络表现的互斥性，作者引入了一个Efficient Channel Attention(ECA)模块，该模块只包括少量的参数，但能够获得优秀的性能。  

通过对SENet的通道注意力拆分研究后，作者发现对于学习通道注意力来说，避免维度缩减是非常重要的，并且适当的跨通道交互能够在有效的减少模型复杂度的前提下保留优秀的表现。因此作者提出了一个不含维度缩减的局部跨通道交互策略，该策略能够有效的通过1维卷积来实现。另外，作者提出了一个自适应选择1D卷积核尺寸的方法，从而决定局部跨通道交互的覆盖范围。作者认为其ECA模型既有效又高效，不管是在参数数量、FLOPs和表现上都表现的非常优秀，并在实例分割、图片分类、目标检测领域均取得了优异的表现。  


作者的idea来源于SENet，SENet采用Squeeze and Excitation模块，即先对channel维度使用GAP，然后使用两个FC层来捕获非线性跨通道交互，作者认为这两个FC层虽然有效，但是其带来的维度缩减会给整个模块的表现带来一定的副作用，且这两个FC层对于捕获所有通道之间的依赖性并不是必要的。因此作者就提出了一个ECA模块，在避免维度缩减 的前提下捕获跨通道的交互。如下图所示，在经过通道维度的GAP之后，ECA模块通过考虑每一个通道的k个邻居来在不进行维度缩减的前提下捕获了局部的跨模态联系。
![92-1](/images/daily paper/92-1.png)  

作者认为这一架构可以通过卷积核尺寸为k的1维卷积来实现，这是由于注意力作用在channel维度上，而用k个channel来关注某一个channel恰好为1×1卷积所实现的功能，因此实现起来非常方便。总结一下，作者的contribution有三：分解了SE模块，证明了避免维度削减，使用恰当的跨通道交互能够有效且高效的学习通道注意力；提出了ECA模块；在多个数据集和任务上证明了其模块比SOTA模型拥有更少的参数和FLOPs数，并能获得更好的表现。  

## Method  



---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

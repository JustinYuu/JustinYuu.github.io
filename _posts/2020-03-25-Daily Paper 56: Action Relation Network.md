---
layout: post
title: "Daily Paper 56: Action Relation Network"
description: "Notes"
categories: [CV-Meta]
tags: [Paper]
redirect_from:
  - /2020/03/25/
---

# Daily Paper 56: Few-shot Action Recognition via Improved Attention with Self-supervision  

## Introduction  

这应该是最近最后一篇paper了，看完这一篇，代表当前主流的图像和视频小样本学习的metric-based方法已经阅读完毕，我就可以滚去写代码了。这篇paper是澳大利亚和牛津合作的，第一版于1月份发表，还在under review状态，是比较新的一篇论文。文章的信息量还是很大的，使用了一个改进版本的时空attention，结合自监督学习，进行视频领域的小样本动作识别。该算法在HMDB51, miniMIT和UCF101数据集上得到了不错的效果。  

作者在Introduction就上图了，用来解释他们的idea，他们想通过自监督的方式来进行数据的增强，顺便提高模型的整合不变性，见下图:  
![56-1](/images/daily paper/56-1.png)  

作者首先在特征空间中使用时空注意力机制，去探测帧中与动作有关联的区域，并highlight关键信息、略过无意义的背景信息。接着作者使用外部的数据增强，训练了一个自监督判别器来识别增强的数据，这样可以提供更多训练数据，并增强模型的鲁棒性，减少不确定性。  

作者引入自监督最重要的目的就是使attention map对时空的增强都具有不变性，不管视频怎么变，如何打乱和整合，关键帧依然是关键帧。但是这用简单的注意力机制是无法完成的，还需要加上一个自监督的loss，用一种类似multi-task的思想来解决这个问题。  

总结一下作者的contribution: 第一，提出了一个用于解决动作识别问题的有效新方法，使用时序和空间注意力机制来定位动作和抑制噪声和背景信息；第二，提出了一个时序和空间的自监督小样本学习方法，通过选择、时空jigsaw来提升了模型的鲁棒性和避免过拟合；第三，作者首先提出了将原始和增强的数据通过自监督注意力机制进行对齐的思想，从而产生了模型的整合不变性。我认为这篇paper最大的贡献还是将自监督引入了小样本学习中，至于注意力机制应该是早就有过的。  

## Method 
 

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

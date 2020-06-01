---
layout: post
title: "Daily Paper 76: CMSA"
description: "Notes"
categories: [CV-Attention]
tags: [Paper]
redirect_from:
  - /2020/05/23/
---

# Daily Paper 76: Cross-Modal Self-Attention Network for Referring Image Segmentation  

## Introduction  

这篇是加拿大的曼尼托巴大学和上海大学发表在CVPR2019上的，主要提出了一个跨模态的自注意力网络，用于基于文本的实例分割。刚好我最近也在搞跨模态的自注意力网络，所以看一下这篇paper。  

跨模态自注意力机制的作用是为了捕获长距离依赖性。之前看过的一些文章，也都想方设法的得到长距离依赖性，比如SE-Net，Non-Local Network，CBAM/BAM，GC-Net等等。这里作者提出了一个跨模态的自注意力模块(CMSA)，能够有效的捕捉语言和视觉特征之间的长距离依赖性，并能够自适应的聚焦在信息量最大的单词和最重要的图片区域上。此外，作者还提出了一个门控的多层次融合模块，从而有选择的整合与图片的不同层次有关的自注意力跨模态特征，该模块能够在不同的层次来控制特征的信息流。作者在4个验证集上验证了该算法的性能，结果显示在该任务上实现了SOTA。  

















---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

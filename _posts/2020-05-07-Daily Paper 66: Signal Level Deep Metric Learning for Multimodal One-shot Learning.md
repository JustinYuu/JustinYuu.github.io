---
layout: post
title: "Daily Paper 66: Signal Level Deep Metric Learning for Multimodal One-shot Learning"
description: "Notes"
categories: [CV-Meta]
tags: [Paper]
redirect_from:
  - /2020/05/07/
---

# Daily Paper 66: Daily Paper 66: Signal Level Deep Metric Learning for Multimodal One-shot Learning  

## Introduction  

今早看arxiv的时候发现四月新挂了两篇小样本动作识别的文章，遂读一下。这篇是德国的科布伦茨-兰道大学的几位学者在四月底挂在arxiv上的，应该还没发出来。他们使用了度量学习的方式来研究多模态的小样本动作识别问题。作者使用度量学习的方法，把动作识别问题转为一个特征空间的最邻近搜索问题，将signals在信号级别上进行编码，使用深度残差CNN进行特征提取，triplet loss进行特征embedding的学习。作者的模型虽然是基于signal提取的，但是在跨模态的表现上超过了NTU RGB+D的baseline，并在UTD-MHAD和Simitate数据集上取得了不错的泛化结果。  

对于小样本学习来说，度量学习和元学习一直是比较火、效果比较好的两个方法。而对于小样本动作识别任务来说，一般都是通过度量学习的角度入手，这里也不例外。该算法在signal level上学习表示，从而能够构建一个能应用于多模态和单模态的one-shot动作识别框架。所谓的跨模态指的是用一个模态来训练embedding，然后在另一个模态上进行动作识别，这个听起来还是挺厉害的。  

作者的主要贡献如下：提出了一个基于学习动作embedding的one-shot动作识别方法，在NTU RGB+D 120数据集上进行了骨架序列的动作识别检验，得到了不错的结果，并泛化到Simitate数据集的motion capturing sequence任务当中。作者还证明了该算法能够有效的应用到跨模态的应用中。  

## Method  







---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

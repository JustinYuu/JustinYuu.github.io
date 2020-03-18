---
layout: post
title: "Daily Paper 49: Two-Stage FSL"
description: "Notes"
categories: [CV-Meta]
tags: [Paper]
redirect_from:
  - /2020/03/18/
---

# Daily Paper 49: A Two-Stage Approach to Few-Shot Learning for Image Recognition  

## Introduction  

这篇paper是普渡大学的学者发在TIP上的，也是去年年底的一篇文章，应该也是比较新的。这篇文章的主要思想是采取了两步法来解决小样本的图像分类问题，其主要步骤是：第一步使用relative feature来将图片映射到一个discriminative space中，然后根据每个类的方差不同，使用一个网络来预测每一个类的方差，最后使用Mahalanobis距离来进行分类，生成每一个类别的mean-class representation；第二步使用一个与类别无关的映射将mean-sample representation映射到对应类别的prototype representation，然后进行分类。  

作者认为他们的contribution有以下几点：提出了一个新的用于物体识别的relative-fearture解释器，并建立了一个用于学习类别方差的框架，从而计算和类别prototypes之间的Mahalanobis距离，还提出了一个新的训练流水线，从而学习一个与类别无关的从class-mean representation到calss prototype的转换关系。总结一下，作者所提出的创新其实就是将普遍的metric learning的训练方式多加了一个stage，即图片到prototype不是一步到位的，而是有一个叫做mean-class representation的中间状态，这个中间状态究竟有什么用，还要看后续的解释。  

## Method  

作者认为该方法同时结合了metric learning和meta-learning的元素，这也体现在该方法的two-stage上面。度量学习的stage用来学习绝对和相对的特征集，然后使用Mahalanobis距离来计算测试样例的标签。而使用相对特征的思想来源于迁移学习中的区域自适应算法，区域自适应算法所研究的是将同一类别的标注源区域数据和未标注目标区域数据尽可能的适应起来，从而提高迁移学习的表现。元学习的stage用来学习用于分类的auxiliary knowledge，具体是指从一个sample到对应的类别prototype的转换，整体的算法流程图见下图:  

![1-1](/images/daily paper/48-1.png)  

整个系统在一个相当大的数据集上进行，每一个类别都有大量的数据，从而学习可泛化的知识，在新任务的小样本中，数据量非常小，可以根据之前学习到的泛化数据来实现快速学习。我觉得这个流程更像是迁移学习，整体的训练样本数还是挺多的。但是这种方式有一些问题，新任务的数据可能和之前的预训练数据集的书籍完全不同，这就会出现高维度和高方差的缺点，因此作者提出了relative features，variance estimator和category-agnostic transformation等一系列补充方法。此外由于传统的训练方法无法模拟小样本的训练，所以作者还是采用了小样本学习所一般使用的episodic training策略，这个策略已经很熟悉了，我就不再赘述了。  

### Relative-Feature-Space Representation  

接下来依次介绍几大部分。第一个就是所谓的相关特征relative-feature空间的表示，由于在一个episode中，query和support set加起来的数量也很少，那么对于一个特征空间来说，其维度相对于数据的总量来说显得非常大，这种绝对特征图内部的稀疏性会导致过拟合和泛化性能差。因此作者提出了相对特征空间表示，其维度是和数据的总量性匹配的，也就是说和原始的绝对特征空间相比维度小了很多。我也不明白为啥特征空间太大了就会过拟合了，可能是embedding可嵌入的选择太多了？作者也没有进一步说明。  

该空间的计算是通过计算某一样本与该episode中的所有样本(包括自己)的欧氏距离的平方根得到的，对于nq+ns个样本而言，特征的维度就是nq+ns。







---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

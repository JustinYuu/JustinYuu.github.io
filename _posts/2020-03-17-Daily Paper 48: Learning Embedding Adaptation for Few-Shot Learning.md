---
layout: post
title: "Daily Paper 48: Learning Embedding Adaptation for Few-Shot Learning"
description: "Notes"
categories: [CV-Meta]
tags: [Paper]
redirect_from:
  - /2020/03/17/
---

# Daily Paper 48: Learning Embedding Adaptation for Few-Shot Learning  

## Introduction  

今天这个比较新，主要使用了transformer的思想来处理小样本的分类问题。在度量学习方向上的研究，一般都是直接采用某一种embedding的学习方式，用同样的方式处理所有分类任务，从而进行度量和分类。但是作者认为，每一个任务都是不同的，所要学习到的差异性也是不同的，如果对于所有的任务，都用同一种嵌入函数来处理的话，那么就相当于需要从这一种分类方式中处理多类任务，这显然是比较困难的，结果也肯定会没那么好。作者因此想提出一种task-specific，也就是独立于特定任务的embedding方式，对于每一类任务，都学习到最好分辨的embedding，再用度量学习的方式进行分类，作者认为结果会好一些。作者采用的方式就是Transformer，在小样本的分类上得到了STOA的效果。  

## Method  

作者首先介绍了传统的小样本学习方式。作者将使用的数据集称作D<sup>S</sup>。该数据集的目的是为了学习分类器f，一般需要从数据集S中取出多个样本，组成多个M-way N-shot的小样本学习任务，这里所有的类别都是support set中见过的，比如matching network等算法中都采用这种方式，然后尽可能的降低损失，优化算法。所谓的分类器f，作者认为可以分为两个部分，第一个就是embedding function，将图片映射到一个特征空间中，第二个是最邻近分类器，根据特征空间中向量的距离来判断类别，这也是metric learning的标准操作，作者将这种方式称作task-agnostic，因为不管是什么子任务，特征空间都是相同弄的。  

接下来是作者的task-specific方式，这里的重点模块叫做FEAT(Few-Shot Embedding Adaptation Transformer)。作者认为主要思想在于新增了一个adaption step适应步，在这一步中，得到的embedding被transform了，这里的转换是端对端的函数，输入的一般是无须的包或者集合，输出的是优化后的embedding，整个过程是permutation-invariant顺序不变的，下图是整个算法的流程图。  

![1-1](/images/daily paper/LEAFSL-1.png)  



---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

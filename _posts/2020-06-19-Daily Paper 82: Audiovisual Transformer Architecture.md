---
layout: post
title: "Daily Paper 82: Audiovisual Transformer Architecture"
description: "Notes"
categories: [MMML-Transformer]
tags: [Paper]
redirect_from:
  - /2020/06/19/
---

# Daily Paper 82: Audiovisual Transformer Architecture for Large-Scale Classification and Synchronization of Weakly Labeled Audio Events  

## Introduction  

这篇文章是KU Leuven发表在MM2019上的，也是一篇视听多模态transformer的文章，应用的任务是弱标注音频事件的广尺度分类和对齐，该任务严格来讲应该属于一个声学任务，这里借用了视觉信息帮助定位。作者将transformer做了一些改良，在该任务上实现了SOTA的表现。  

Audio event classification任务一般使用AudioSet数据集或其子集，也有专门的比赛DCASE，不过传统的audio event classification更多的是单模态模型，即只利用了声学模态的信息。作者进一步想到，最近很火的transformer就是把两个不同的language放到一起处理，和这里的两个不同模态类似，也许能把transformer应用到多模态的领域，因此就产生了使用transformer来处理audiovisual classification of environmental events任务的想法。  

## Method  

首先吐槽一下，MM本来就这么短的篇幅，作者居然还介绍了一整页的transformer。。对transformer结构图，以及对多头注意力机制的诠释我全部略过。直接看作者的修改版，示意图如下：  
![82-1](/images/daily paper/82-1.png)  

这里的修改和我想象中的还是有区别的。整个流程采用encoder-decoder架构，positional encoding是可选的，一个模态的信息进入encoder，内部的架构和transformer的encoder几乎一样，decoder的输入是另一个模态信息，在decoder内部进行self-attention+multimodal-attention的操作，multimodal的K和V是encoder的输出。最后通过一个sigmoid层和aggregation模块来得到最后的预测值。  

第一眼看上去貌似和ViLBERT这些Bi-Modal Transformer不太一样，但是仔细想一想，其实还是一个意思，这里的encoder-decoder架构其实就是一个two-stream网络，第一个模态和第二个模态都经过self-attention处理后，使用一个multi-head attention  fusion起来，那么这里的multi-head attention其实起到的是fusion的作用，我认为这是这篇文章的模型和其他Bi-modal Transformer模型最大的区别。  

接下来补一些小细节。首先是positional encoding之前的线性层，这个是作者加上去的，原因是视觉信息和音频信息的尺寸不同，这个很好理解。其次是output层的修改，由于该任务的特殊性导致预测类不是互斥的，一个时间段可以有多种动作同时发生，因此无法使用softmax进行分类，而是改成了使用sigmoid层。此外后面还有一个整合层，这是因为该任务要求一个视听片段只需要一个概率向量来完成单标签的预测，所以需要额外添加一个模块，来让frame-level的预测分数转换为clip-level的预测结果，这很自然的会想到用平均或最大池化来实现。  

作者还把attention函数做了一些调整，在transformer中的scaled dot-product attention中，有一个softmax，但是作者可能不太喜欢softmax，就又做了一系列实验，来探究不同的替代函数带来的影响，替代函数包括sigmoid和normalized sigmoid，后者的公式如下:  
![82-2](/images/daily paper/82-2.png)  

最后，作者认为positional encoding是可选的，因为作者不清楚数据的序列信息到底有多重要，于是做了些实验来验证positional encoding的效果，具体结果见下文。  

## Experiment  

实验细节全部略过，直接放结果:  
![82-3](/images/daily paper/82-3.png)  

结果还是有意思的，这个结果其实就是一个ablation study，首先看positional encoding：结果显示视频模态的positional encoding能够微弱的提升表现，但是单一音频模态的positional encoding却明显有害处，而对于多模态，给视觉信息encoding会微弱的提升表现，对双模态都进行encoding会微弱的减弱表现。那么虽然作者没有进一步总结，但是我觉得已经很清楚了，positional encoding只对视觉信息有效，对声学信息无效甚至有反作用。  

其次是aggegration method和attention function。结果显示平均池化比最大池化好，不同的函数对表现的影响其实不大，这些实验我觉得没多大意思，自己强行搞了些想法去验证，验证完了发现都一样，怎么看都像是凑数的。  

那么和之前的结果对比一下，如下图:  
![82-4](/images/daily paper/82-4.png)  
![82-5](/images/daily paper/82-5.png)  

效果的提升还是相当明显的，不过这个baseline看起来都好弱。。  

可视化结果就不放了。  

## Conclusion  

总结一下，在2019年年初的时间点，做出这个工作算是紧跟潮流了，idea还算新颖，结果不错，实验也相当详细，也有丰富的可视化结果，应该算是一篇比较标致的文章。这里面一些细节可以借鉴一下，比如把multimodal transformer只作为fusion，以及对positional encoding的看法等等。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

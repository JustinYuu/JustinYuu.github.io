---
layout: post
title: "Daily Paper 86: Multimodal Matching Transformer for Live Commenting"
description: "Notes"
categories: [MMML-Transformer]
tags: [Paper]
redirect_from:
  - /2020/06/30/
---

# Daily Paper 86: Multimodal Matching Transformer for Live Commenting  

## Introduction  

这篇是哈工大和MSRA于二月底挂在Arxiv上的，使用了一个多模态匹配Transformer用于实时评论任务。实时评论任务我一直没想明白有啥用处，但是我理解它作为评价video-to-text生成水平的一个评价指标。目前该任务一般使用encoder-to-decoder架构，但是作者认为这些方法并没有显式的对视频和评论之间的关系建模，因此很容易去生成一些与视频并无关系但是非常流行的评论。这篇文章作者就力求提升实时评论和视频之间的相关性，这是通过对不同模态之间的跨模态关联建模来完成的，使用transformer框架来实现跨模态互动。作者使用其模型在公开的实时评论数据集上获得了SOTA的表现。  

## Method  

这篇文章先介绍了一下Transformer架构，这里我就不多说了，毕竟这几篇全都是Transformer的改进，已经非常熟悉了。首先对该任务进行一下定义，给定一个视频V和一个时间戳t，自动实时评论系统系统视图去从一个待选集Y中选择一个评论y<sub>\*</sub>，该评论被认为是在时间戳t附近的视频片段中最为相关的评论，考虑的要素包括周围的评论C，视频部分F和音频部分A，那么整个任务可以看做是从多模态的语义空间内找最为相关的视频片段，公式如下:  
![86-1](/images/daily paper/86-1.png)  

其中S是进行相似度测量的模型。  

整个网络的架构如下:  
![86-2](/images/daily paper/86-2.png)  

网络可以分为三部分，第一部分是encoder层，将多个模态的信息和candidate评论转为向量；第二部分是matching layer，该部分迭代的对每一个模态的信息都学习一个attention-aware表示；第三部分是prediction layer，也就是decoder，输出一个video clip和comment的匹配度，从而选出匹配度更高的评论。  

接下来详细介绍一下这三层。首先是Encoder，对于comment，首先把接近时间戳t的N<sub>c</sub>个评论concat起来生成一个大的评论C，然后使用一个embedding表M来将这个大comment C转化为向量。同样的把candidate comment y也转为向量。对于视频的视觉信息，使用预训练的ResNet-18来提取特征。对于音频，首先把5s的音频片段分解为5个音频帧集，然后使用GRU来对每一个集进行编码，将最后一个隐藏层的集合作为音频片段的表示。此外对于每一个模态都进行positional encoding。  

接下来是matching layer，这也是最为精华的部分。从上图可以看出是以SA+CMA为骨架的架构实现的，CMA的q是当前模态，k和v是其他模态。得到y和c,f,a三个模态分别交叉的特征后，再使用一个MLP作为融合门，将其作为权重进行加和，公式如下:  
![86-3](/images/daily paper/86-3.png)  
最后再将得到的结果通往一个前向FNN中，也就是一个双层的MLP得到最终的结果。  

最后是预测层，首先使用一个加权的池化层来将上个层的输出转化为一个固定长度的向量，公式如下:  
![86-4](/images/daily paper/86-4.png)  

上式是对于评论y，对于C,F和A来说同样如此，接着使用一个融合的gate来将这三个特征转为联合的特征，公式如下:  
![86-5](/images/daily paper/86-5.png)  

最后再使用一个余弦相似度来衡量联合表示V<sub>context</sub>与V<sub>y</sub>之间的相似度。  

训练的过程使用max-margin loss，公式如下:  
![86-6](/images/daily paper/86-6.png)  

## Experiment  

评价指标使用Recall@k,Mean Recall和Mean Reciprocal Rank，结果如下：  
![86-7](/images/daily paper/86-7.png)  

作者还放了一个例子，因为是HIT的文章，所以用的中文词汇，看起来还是挺有趣的:  
![86-8](/images/daily paper/86-8.png)  

## Conclusion  

我认为这篇文章的创新点和之前几篇的并无不同，只是换了个任务而已。而由于换了个任务，所以没法使用经典的encoder-decoder架构，所以使用了一个matching network的思想来计算相似度。这篇文章值得借鉴的应该是fusion模块采用的思想。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

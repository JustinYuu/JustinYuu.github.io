---
layout: post
title: "Daily Paper 88: A Novel Attention-based Aggregation Fusion Function to Combine Vision and Language"
description: "Notes"
categories: [NLP]
tags: [Paper]
redirect_from:
  - /2020/07/07/
---

# Daily Paper 88: A Novel Attention-based Aggregation Fusion Function to Combine Vision and Language  

## Introduction  

这篇文章是由几个意大利人在四月底挂在arxiv上的，主要研究视觉和语言的联合表示。作者认为由于图片和文本都能用集合或者序列的方式来表示，比如多个区域和多个单词，那么就需要一个适当的函数来将一系列编码后的元素变成一个单独的response，比如一个分类结果或者相似度分数。在这篇文章中，作者提出了一个新的全部由注意力组合而成的reduction function。具体来讲，作者提出的方法对每一个模态的每一个元素，使用一个跨模态注意力的变体来计算了一系列分数，进行了一个可学习的跨模态reduction，这个分数可以用于分类和排名。作者在image-text匹配和VQA两个任务上进行了测试，在COCO和VQA 2.0数据集上获得了和当前SOTA相当的表现。  

## Method  

架构的示意图如下：  
![88-1](/images/daily paper/88-1.png)  

从上图可以看出，首先使用region encoder和 word encoder对视觉和听觉的信息进行预处理，然后使用CMA+SA进行跨模态特征提取，最后使用基于注意力的融合函数对齐两个模态的信息并分别进行降维。  

那么重点就在attention-based aggregation function上，作者的整合函数基于放缩点乘注意力机制，在每一个头内，自注意力的qkv都是一样的，而跨模态注意力的q是当前模态，kv是另一模态。这里作者定义 了一个Score Attention操作，对于query序列的每一个元素都计算一个相关性分数，代表当前query与其他模态的kv之间的关联性，公式如下：  
![88-2](/images/daily paper/88-2.png)  

可见就是多了一个FC层，对于每一个query都能得到一个分数值。那么给定一系列模态A的输入向量X，再给定另一系列模态B的输入向量Z，就能获得一个最终的X和Z之间的关系，可以使用加权得到，公式如下:  
![88-3](/images/daily paper/88-3.png)  

这只是从Z到X的模态对应，而对于Z，也使用相同的方法得到X到Z的模态关系。  

总之，这一函数可以被看作一个模态关联性计算函数，并且每一个模态的压缩表示可以捕获一个对于输入的全局视角，从而更加关注那些跨模态交互更为重要的部分。此外，作者认为这个整合函数可以平行的在不同的qkv投影中使用多次，从而生成多个输出向量，这原则上可以获得一个更为自由的表示。作者对整合函数使用的次数进行了实验，并用一个超参数k来定义，当k>1的时候，最终的输出由这些输出的平均值得到。  

作者还交代了一下视觉语义模型的细节。该模型可以同时用于图片-文本匹配任务和VQA任务，作者用一个双向的GRU来处理text，用一个线性的投影矩阵来处理图像区域，这里应该是用CNN提取好了之后再用一个FC处理一下。  

最后是训练的细节。在VQA任务中，经过整合函数操作后，两个vector会被concat起来，然后通往一个FC层中得到最终的分类结果，此外还在reduction操作和最终的concat中间添加了一个position-wise feed-forward层。训练的时候使用多标签的二元交叉熵误差进行训练，该实验并没有使用任何数据增强策略，也没有使用外部的数据。在跨模态检索任务中，使用余弦相似度来衡量其相似度，在训练的时候使用hinge-based triplet ranking loss，函数公式如下:  
![88-4](/images/daily paper/88-4.png)  

## Experiment  

最后放一下结果，在VQA的表现如下:  
![88-5](/images/daily paper/88-5.png)  

检索的表现如下:  
![88-6](/images/daily paper/88-6.png)  

## Conclusion  

这篇文章提出的方式还是挺有趣的，我觉得这个融合方法可以搬到目前的任务上来，不过文章中没有解释清楚的是为什么跨模态+SA+相同的跨模态注意力融合方式会优于跨模态+SA+concat作为融合方式，也就是说有效性究竟是前面的CMA+SA还是后面的融合方式呢？好像并未看到类似的ablation study结果。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

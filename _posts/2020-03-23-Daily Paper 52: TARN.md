---
layout: post
title: "Daily Paper 52: TARN"
description: "Notes"
categories: [CV-Meta]
tags: [Paper]
redirect_from:
  - /2020/03/23/
---

# Daily Paper 52: TARN: Temporal Attentive Relation Network for Few-Shot and Zero-shot Action Recognition  

## Introduction  

这篇文章是queen mary的三位学者发表在BMVC2019上的，我最近越来越发现BMVC的质量还是挺不错的，虽然是个小会议，但是至少我看到的文章还是写的挺好的，不知道为啥不投CVPR和ICCV。  

这篇文章中作者主要提出了一个新的时序注意力关系网络Temporal Attentive Relation Network(TARN)，用来解决小样本和零样本的动作识别问题。作者认为其核心思想是使用了元学习方法来学习比较多种时序长度的表示，也就是说任意两个不同长度的视频，甚至一个视频和一个类似于词向量的语义表示(用于零样本学习)，都可以进行比较。不过这个我感觉novelty不是很足，好像大家都可以做到。和其他的工作相比，作者利用了注意力机制来实现时序对齐，并学习了一个视频碎片级别的对齐表示的深度距离量化方法。作者使用episode训练方法，用一种端到端的方式来训练网络，结果实现了动作识别的STOA，在零样本学习的效果也很不错。  

## Model  

### Overview  

首先来看整体的流程图，如下图:  
![52-1](/images/daily paper/52-1.png)  
整个网络由两部分组成，embeddings module和relation module，然后输出一个relation分数，通过softmax完成分类，整体流程还是标准的metirc-based method的步骤。他们主要的contribution就是在于使用了注意力机制将视频内部分割为不同的segment，然后将视频的比较转化为segment集合的比较。  

### Embedding Module  

FSL和ZSL使用的方式略有不同，因为ZSL需要生成semantic embeddings，而FSL只需要生成Video embeddings就可以了。对于Video Embedding，还是使用的最为基础的方法，即一个预训练的C3D网络和一个Bi-directional GRU。C3D的作用是提取segment中的时空local features，双向GRU是为了使用local features学习到globally-aware fetuares，也就是将时间步考虑在类，顺便起到了降维C3D特征的作用。  

ZSL首先要将query video中的segments的图像提取成为视觉表示，然后再与sample set classes的语义表示进行比较。初始的语义信息需要被编码到一个能够和视频片段进行比较的特征丰富的与类有关的特征表示，因此也需要与视觉表示有相同的维度。所以需要通过一个2层的FC层来处理。  

这篇paper的重点不是embeddings module，所以这一部分没有什么contribution，基本上是沿用之前的工作。  

### Relation Module  

作者首先介绍FSL的方法，然后再扩展到ZSL。作者首先使用一个C3D和GRU来提取视频特征。由于作者使用relation network的方法，需要将query video与support set中的所有video逐一比对。对于每一对视频(Q,S)，都使用一个segment-by-segment注意力层将视频中的片段对齐，这是为了使不同长度的视频能够进行比较。接下来，将query中的segment和与之对齐的sample segment比较，然后将不同片段之间的比较结果放到DNN中，学习一个用于比较视频的深度metric，然后输出每一对的realation score，注意这个score是video间的，最后使用一个softmax层将关系分数转化为相对于全部类别的概率。  

接下来看细节。segment-by-segment attention用来对齐视频片段。这个思想来自于NLP中的word-by-word attention，用来对齐两个长度不同的句子。类似的，作者将word-by-word attention移花接木，搞到了segment-embeddings中，变成了segment-by-segment attention。具体操作方式如下：对于一个N×d的support video和M×d的query video，分别有N和M个segments，那么使用下列公式 A = softmax((SW) + b⊗e<sub>N</sub>)Q<sup>T</sup>), H = A<sup>T</sup>S. W和b是要学习的权重，b⊗e<sub>N</sub>代表broadcast b从1×d到N×d，说白了还是shift and scale。A是最后的注意力权重矩阵，H是S对齐后的版本，H的每一行都是S的segment-embeddings的加权和，代表着S中与对应的Q的行向量最相关的部分，Q和H的行向量作为后续比较层的输入，两者的尺寸都是M×d。  

然后是deep metric learning部分，也就是relation network的思想，使用一个DNN来学习深度相似度/距离指标，这里由于是视频，所以需要通过Q和H的M个segments综合比较。由于输入是M维的，输出的结果也是M维的，作者认为这种将整体的视频拆分成多个片段的方式是有效的。对于FSL，作者使用了一个单向GRU来学习不同片段比较结果的时序信息，用一个FC层来得到最后的关系分数。




---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

---
layout: post
title: "Daily Paper 84: Multimodal Transformer Network"
description: "Notes"
categories: [MMML-Transformer]
tags: [Paper]
redirect_from:
  - /2020/06/29/
---

# Daily Paper 85: Multimodal Transformer Networks for End-to-End Video-Grounded Dialogue Systems  

## Introduction  

这几天放假划了几天水，所以没怎么更新。这篇文章提出了上一篇所使用的Multimodal Transformer Networks，由新加坡管理大学作为第一作者发表在ACL2019上。该文章主要聚焦Video-Grounded Dialogue Systems(VGDS)任务，即给定一段视频，根据视觉和音频信息来生成一段对话。该任务的困难之处在于视频的特征空间分布在多个图片帧上，使得获取语义信息比较困难，此外一个dialogue agent必须接收和处理多个模态的信息来获得一个多模态的联合表示。目前为止大多数方法都聚焦在RNN和seq2seq架构上，该架构并不能够有效的捕捉复杂的长距离依赖性，因此自然而然的就想到了transformer，也就提出了多模态Transformer模型。此外作者也提出了一个query-aware attention，通过一个自编码器来从非文本模态中提取query-aware特征。作者还提出了一个训练步骤，来模拟token级别的解码，从而提升生成的回答的准确性。该任务和之前看到的DTSC7几乎是一样的，训练集也是用的DSTC7数据集，在当时的DSTC7比赛中获得了SOTA的表现。  

## Method  

其实这篇文章和上篇文章的模型一样，任务一样，数据集一样，唯一的区别在于这篇文章详细的介绍了他们所创造的MTN网络，而上一篇文章详细的介绍了他们是如何应用MTN网路，通过一些trick和修改来得到好的结果的。那么这篇文章的重点也就自然而然的放在了MTN网络的本身，而不再介绍任务、数据集和实验细节部分。  

![85-1](/images/daily paper/85-1.png)  

上图是MTN网络的整体流程图，可以看出整个网络分为三个部分：Encoder, Decoder和Query-Aware Auto-Encoder(QAE)。为了简略，上图把Feed Forward, Residual Connection和Layer Normalization省略了，下面详细的介绍每一部分。  

首先是encoder，encoder层的作用是将文本序列和输入的图片处理成为连续的表示，这其实也是地球人都知道的东西了。这里采用了Positional Encoding，作用是分别在token和视频帧级别上注入序列信息。那么详细来看，encoder其实可以再分为两部分，文本序列encoder和视频encoder，示意图如下:  
![85-2](/images/daily paper/85-2.png)  

对于文本序列encoder，首先进行token-level embedding，其次进行positional encoding，最后通过层间均一化得到最终的encoder结果。对于视频encoder，首先通过一个n帧的滑动窗口和一个预训练的网络进行视频特征提取，得到视频/音频双模态特征，接下来通往一个FC+ReLU来转换维度，进行Positional Encoding后通过层间均一化得到最终结果。两个encoder的Transformer都使用的Attention is all you need中的positional encoding。从上图中可以看出，这里的encoder和transformer有很大的不同，没有多个自注意力层的堆叠，而是只应用了层间均一化处理embedding。  

接下来是decoder。decoder的作用是给定原序列连续表示z<sub>s</sub>和偏移目标序列z<sub>t</sub>，生成一个输出序列。decoder由N个相同的层来组成，每一个层都有4+\|\|M\|\|个子层，每一个子层都对独立的编码输入做注意力，输入有五个，分别是偏移目标序列、对话历史、视频caption、用户query和非文本视频特征（视频、音频）。每一个子层都由一个多头注意力机制和一个position-wise前向传播层组成，每一个前向传播层由两个FC+ReLU组成，并采用了残差结构和层间均一化。  

除了encoder-decoder架构之外，作者还提出了一个Auto-Encoder Layers,该层的提出是因为多头注意力允许在不同的输入组件上进行动态注意力，那么输入的query和非文本视频特征之间的必要互动并没有完全实现。由于采用了残差结构，并将视频注意力模块放在了decoder层的最后，那么视频特征的attention可能并不是最好的，因此作者猜测在视频特征上添加一个query-aware attention作为一个独立的组件可能会提升表现。作者设置该模块，让模型对query-related的视频特征以无监督的方式加强注意力，auto-encoder也由N个层堆叠而成，每一个层都含有一个query self-attention和视频特征身上的query-aware attention，也就是SA+CMA，总层数是1+\|\|M\|\|。不管是4+M也好，1+M也好，从示意图上可以看出都是串联的，我其实觉得并联之后用fusion的方式处理更好一些。对于CMA，音频和视频特征分别作为k和v，与作为q的经过SA处理过的特征z进行CMA，公式如下：  
![85-3](/images/daily paper/85-3.png)  

在decoder中，通过一个线性转换层+Softmax来生成了下一个token的词汇表概率，而在auto-encoder中，也使用了同样的架构来重新生成了query序列。作者将原序列embedding，输出embedding和softmax之前的线性转换的权重矩阵分开训练。和训练过程不同的是，在测试的时候，decoding仍是一个自回归过程，也就是说decoder以token-by-token的方式生成句子。作者试图去在训练的过程中去模拟这一过程，方式是给定一个概率p，代表有p的概率使用随机选择的位置i来对原序列进行裁剪，只保留位置i之前的部分作为target序列。作者使用这个方式来减少了训练和测试过程中输入到decoder的信息的不匹配，从而提升了生成response的性能。作者只对decoder中的目标序列这么处理了，对query auto-encoder中的目标序列并未做处理。  

实验的loss如下：  
![85-4](/images/daily paper/85-4.png)  

## Experiment  

实验细节就不放了，超参数放一下，作者有两套超参数，如下图所示:  
![85-5](/images/daily paper/85-5.png)  

可见网络的规模还是不小的。结果对比如下，可见实现了SOTA。  
![85-6](/images/daily paper/85-6.png)  

作者做了大量的消融实验，以及不同超参数和网络架构给实验结果带来的影响，这里略过不表。  

## Conclusion  

MTN和ViLBERT属于同一时期的文章，都是将Transformer应用到了多模态领域上，应用的方式宏观思想基本相同，微观细节略有差异，值得借鉴到更多跨模态任务上。此外，这篇文章和之前的文章都提到了使用自监督或无监督的方式来联合训练以提高最终性能的trick，我认为值得参考。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

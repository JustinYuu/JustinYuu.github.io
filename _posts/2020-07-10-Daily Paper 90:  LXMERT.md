---
layout: post
title: "Daily Paper 90: LXMERT: Learning Cross-Modality Encoder Representations from Transformers"
description: "Notes"
categories: [MMML-Transformer]
tags: [Paper]
redirect_from:
  - /2020/07/10/
---

# Daily Paper 90: LXMERT: Learning Cross-Modality Encoder Representations from Transformers  

## Introduction  

这篇文章是昨天看上一篇的时候看到的，也是一个多模态的transformer，由北卡的Hao Tan和Mohit Bansal发表在EMNLP2019上。LXMERT全称为Learning Cross-Modality Encoder Representations from Transformers，也就是用Transformer来学习跨模态encoder表示。具体来讲，作者建立了一个大的transformer，该模型主要由三部分组成：一个object relationship encoder，一个language encoder和一个跨模态的encoder，之后在五个预训练任务上进行了预训练，分别是masked language modeling, masked object prediction(特征回归和标签分类), cross-modality matching和image question answering。这些任务能够学习到跨模态和单模态的信息。经过预训练之后再进行fine-tune，从而在两个VQA数据集上得到了SOTA的表现。此外，作者发现他们的预训练跨模态模型可以泛化到一个更有挑战的视觉理解任务NLVR上，并能有效的提升该任务的SOTA22%，可以说是提升相当明显了。  

## Model Architecture  

首先看一下整体的网络流程图，如下图所示:  
![90-1](/images/daily paper/90-1.png)  

### Input Embeddings  

LXMERT的输入层将输入的图像和句子转化为了两个特征序列：词语级别的语句embedding和物体级别的图像embedding。这些embedding特征将会被后续的网络层处理。  

对于句子来说，首先将一个句子分成若干个单词，使用WordPiece tokenizer进行处理，接下来通过embedding层进行处理得到词嵌入，再经过index-aware word embedding处理和层间均一化得到最终的word-level sentence embeddings。  

对于图像来说，这里并没有使用卷积层的输出，而是使用了图像中检测到的物体的特征作为输入特征。具体来说，目标检测器从给定的图片中检测到m个物体，每一个物体都由其位置特征和其2048维的RoI特征组成。此外还添加了两个FC层，从而能够得到positional-aware embedding，公式如下:  
![90-2](/images/daily paper/90-2.png)  

由于图嵌入层和后续的注意力层都与输入的绝对序列无关，因此检测到的目标的顺序并没有显式的指定。  

### Encoders  

encoder主要基于注意力机制和跨模态注意力机制。单模态encoder基于自注意力+Feed Forward层，而多模态encoder基于两个自注意力层、一个双向跨模态子层和两个feed-forward层，顺序是CMA+SA+FF，公式如下:  
![90-3](/images/daily paper/90-3.png)  
![90-4](/images/daily paper/90-4.png)  

此外在每一层都和transformer类似采用残差结构。  

### Output Representations  

跨模态模型会产生三个输出，分别是语言输出，视觉输出和跨模态输出。其中视觉和语言的输出是由跨模态encoder生成的序列，而跨模态输出是在sentence words前面加了一个特殊的token\[CLS]，代表对应的图像特征，从而组成了跨模态的输出。  

## Pre-Training Strategies  

为了更好的初始化并理解视觉和语言之间的联系，作者在一个大的整合数据集上进行了不同模态预训练任务上的的训练。由于涉及到多个任务，而且这些任务也不是我想看的重点，就不一一详细介绍了，下图是整体的预训练流程图:  
![90-5](/images/daily paper/90-5.png)  

## Experiment  

在VQA和GQA数据集上测试的结果如下:  
![90-6](/images/daily paper/90-6.png)  

剩下的大量ablation study就不放结果了。  

## Conclusion  

这篇文章的内容很多，而我充其量只是介绍了整个网络框架这一部分，所以写的很不全面。但是这篇文章在结构性上给我的启示比较多：第一，为什么BERT也好，ViLBERT也好，LXMERT也好，都有多个注意力层的堆叠，而在我的任务中始终无法通过堆叠多层来提高性能？我觉得可以从超参数上找一些原因；第二，SA+CMA还是CMA+SA？之前看到过SA+CMA的例子，但是这几篇文章都是采用的CMA+SA，我觉得可以分别尝试一下；第三，Feed Forward层的使用，我之前是SA+FF+CMA+FF，但是这里采用的是CMA+SA+FF，即只采用一个FF，我认为也是可以尝试的。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

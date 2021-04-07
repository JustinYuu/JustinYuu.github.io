---
layout: post
title: "Daily Paper 97: ViT"
description: "Notes"
categories: [CV-Vision-Transformer]
tags: [Paper]
redirect_from:
  - /2021/04/07/
---

# Daily Paper 97: AN IMAGE IS WORTH 16X16 WORDS: TRANSFORMERS FOR IMAGE RECOGNITION AT SCALE  

## Introduction  

ViT被许多人认为是视觉Transformer领域的开山之作，不同于之前的Visual Transformer, ViT最大的特点就是用纯Transformer架构来解决了视觉的分类问题，并取得了超越CNN架构的性能，并成为了之后重要的baseline。ViT的一大创新，是把图片分成了多个大小相同的patches，然后输入到transformer架构中。在数据集规模较小的时候，由于transformer架构并没有卷积操作的一些归纳偏置，比如translation equivariance and locality，因此效果并没有resnet好，但是当数据集的规模相当大时(14M-300M张图片)，纯transformer架构(ViT)能够达到超越CNN架构的表现。  

## Method  

其实ViT的方法很简单，整个架构如下图所示:  
![97-1](/images/daily paper/97-1.png)  

整个框架可以分为几部分：将图片分成多个patch，进行线性变换并进行位置编码，导入Transformer进行处理，使用MLP进行分类。  

对于这四步，可以用下图四个公式来表示。首先，将图片分成N个patch，每个patch的大小是P×P，为了保持每层patch数量的统一，再用一个独立的FC层将N映射为D，使得每层的patch数量都是D。BERT等NLP架构需要有一个\[class\] token，这里也添加了一个可供学习的embedding作为分类的token，这样最后得到的除了有一系列patch features，还有头部的classification token，只需要再使用一个MLP head即可完成分类，MLP head在训练的时候用一个2层的MLP实现，测试的时候改成1个独立的FC层。对于位置编码，作者使用的就是原transformer中的1维位置编码方式。对于transformer，作者使用的是Vaswani的原始transformer block，只不过非线性层使用的是BERT中的GELU。  
![97-2](/images/daily paper/97-2.png)  

除此之外，作者还建立了一个用CNN进行特征提取的ViT模型作为对照，称为hybrid architecture，这一架构也是类似于Visual Transformer的CNN+Transformer架构。  

## Experiments  

method部分的介绍其实很简单，文章的大部分重点都在实验部分。作者搞了三个模型，分别是Base, Large, Huge级别。不同点如下图的表1所示，区别在于层数、隐藏层尺寸、MLP的维度和注意力的头数。结果可以看到在JFT-300M上预训练的模型超越了同等级的ResNet baseline。剩下还有一大堆可视化和参数量的实验，就不放结果了。  
![97-3](/images/daily paper/97-3.png)  

## Conclusion  

这篇文章其实主要提出了一个新的范式，在方法的创新度上是很有限的。很好的结果也是通过巨大的训练集和模型参数量所实现的。在参数量和训练集规模上还是亟待优化的。此外，作者也提出了，对于自监督的效果不是很好，而监督学习的代价又很大，所以在自监督的条件下也有可优化的场景。最后，这一框架只应用在最基础的视觉任务-图像分类上，对于其他的任务还可以用新的transformer架构来应用。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

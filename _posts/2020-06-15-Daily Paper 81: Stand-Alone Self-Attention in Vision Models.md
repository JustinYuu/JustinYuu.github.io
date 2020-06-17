---
layout: post
title: "Daily Paper 81: Stand-Alone Self-Attention in Vision Models"
description: "Notes"
categories: [CV-Attention]
tags: [Paper]
redirect_from:
  - /2020/06/15/
---

# Daily Paper 81: Stand-Alone Self-Attention in Vision Models  

## Introduction  

这篇是Google Brain的Ramachandran于19年6月份挂在arxiv上的，ICLR2020的attention-CNN也是根据这篇文章进行的更改和补充。卷积一直都是计算机视觉领域的一个基础模块，但是众所周知卷积是无法获得长距离依赖性的，那么为了获得长距离依赖性，也有众多学者进行了许多研究，这些研究更多的是将传统的卷积层和基于内容的互动结合起来，比如自注意力机制和non-local均值，从而在很多视觉任务上获得了更好的表现。不管是non-local，还是自注意力，其实核心思想都是注意力，那么作者就想了，注意力这么强大的东西，真的只能作为卷积层的辅助吗？换言之，能不能只用注意力层，来完成之前卷积层起到的效果？其实这是一个非常困难的任务，首先需要搞懂卷积层的原理，然后尽可能的使用注意力机制来模仿卷积层，并试图寻找全新的超参数来得到媲美甚至超越卷积层的效果。经过一系列实验，作者认为自注意力可以有效的作为一个独立层进行特征信息的处理，将ResNet的所有空间卷积层换成自注意力机制后，在图像分类任务的FLOPs和参数数量上分别减少了12%和29%，并获得了更好的效果，在COCO目标检测数据集上减少了39%的FLOPs和34%的参数，并获得了和RetinaNet相同的mAP值。总之，作者用实验证明了单独的自注意力层可以更高效的获得和卷积层相同的表现。  

## Method  

### Stand-Alone self-attention layer  

首先看一下这个独立的自注意力层究竟长啥样。和卷积层类似，给定一个像素点x<sub>i,j</sub>，首先提取了距离该位置为k以内的局部区域，记作memory block记忆块。这是一种局部注意力的形式，和之前一些对所有像素求全局注意力不同。全局注意力只能用于有效的空间下采样之后，因为它的计算量太大了(比如前两天看的non-local network)，在全注意力模型中局限性太大。注意力采用单头注意力形式，计算最终的像素输出的公式如下：  
![81-1](/images/daily paper/81-1.png)  

其中query是Q矩阵和像素点的乘积，而key和value分别是K矩阵和V矩阵与局部区域内所有点的乘积，注意力是点乘注意力。作者经过实验发现，使用多头注意力形式可以学习到更多不同的输入特征表示，这可以把像素特征depthwise到N个groups中，然后在每一个group中独立的进行单头注意力机制的计算，这是的转换矩阵参数都是不同的，最终将每个group得到的表示concat到一起，得到最终的输出y<sub>ij</sub>，整个过程的流程图如下：  
![81-2](/images/daily paper/81-2.png)  

到目前为止，介绍的attention还没有positional encoding，这就使得整个模型permutation equivariant，这对一些视觉任务来说限制了表达的性能。Transformer中使用了正弦embedding，但是作者认为relative positional embeddings更好一点。relative positional encoding在ICLR2020那篇介绍过，在空间位置上定义行和列的两个offset，代表局部区域内的当前点和中心点之间的行和列距离，示意图如下:  
![81-3](/images/daily paper/81-3.png)  

这样整个attention的公式就变成了下面的样子：  
![81-4](/iamges/daily paper/81-4.png)  

### Fully Attentional Vision Models  

搞定了注意力模块之后，就可以着手建立一个全注意力网络了，作者将其分为两步：替换空间卷积和替换卷积stem。空间卷积即卷积核尺寸大于1的卷积操作，之所以排除1×1卷积是因为它基本上和FC层的效果是相同的，没有像素的扩展，只对单一像素进行处理。那么作者的替换策略也很简单粗暴，就是把每一个时空卷积层替换为自注意力层，后面跟一个步长为2的2×2平均池化操作，以实现空间下采样操作。也就是说把卷积替换成了自注意力+平均池化两部分，分别完成特征转换和维度变换两部分任务。  

第二步叫做替换Convolutional Stem，所谓的stem，就是CNN的初始层。初始层之所以重要，是因为它在学习边界等局部特征中至关重要，初始层学习的结果会被后面的结果使用，以识别全局物体。那么由于输入的图片一般都比较大，所以stem一般和核心卷积块长得不太一样，更多的是通过空间下采样来聚焦轻量化操作。比如在ResNet中，stem是一个步长为2的7×7的卷积操作后跟一个步长为2得到3×3最大池化。说实话这个stem的作用我之前还真不知道，我只是以为是为了迅速的降维。。  

在stem层中，内容包含RGB像素，这些像素单独看起来是没有具体信息的，并且这些像素有着高度的空间关联性。这一特性使得学习有效的特征，比如边缘探测对于基于内容的机制（比如自注意力）来说很困难。作者的实验也证明了使用上述的自注意力层作为stem层比使用ResNet的卷积stem效果要差，因此需要想一个办法来使用自注意力层来代替卷积stem。  

作者的方法是通过spatially-varying的线性转换以pointwise 1×1卷积的形式注入基于距离的信息，新的value由公式v<sub>ab</sub> = (∑<sub>m</sub>p(a,b,m)W<sub>V</sub><sup>m</sup>)x<sub>ab</sub>得到，其中multiple value矩阵W<sub>V</sub><sup>m</sup>是通过一个参数的凸组合得到的。position dependent factors和卷积类似，都在一个邻域上学习了与像素位置相关的标量权重。然后，stem由具有空间感知的value特征的注意力层组成，后跟最大池化。为了方便操作，注意力感受野与最大池化的窗口对齐。  

## Experiment  

在ImageNet上的结果如下图，Baseline是ResNet。可以看出其实还是Conv-stem的效果好一点，但是Full Attention也有不错的准确率提升，且FLOPs和参数数量明显减少。  
![81-5](/iamges/daily paper/81-5.png)  

在COCO上的结果如下图，Baseline是RetinaNet，可见和Baseline的效果几乎相同的同时，大幅减少了FLOPs和参数数量。  
![81-6](/iamges/daily paper/81-6.png)  

ablation study略过。  

## Conclusion  

这篇文章的贡献还是挺大的，首先提出了一个全注意力网络的模型，然后做实验证明了在参数和FLOPs明显小于卷积网络的前提下，能在目标检测和分类两大主流CV任务上取得和Baseline媲美甚至更好的表现，这篇文章可以和ICLR2020的attention-CNN结合起来看，这篇提供了最初的模型框架和实验支撑，后者提供了丰富的理论证明，来严格论证了attention代替卷积的可行性和有效性。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

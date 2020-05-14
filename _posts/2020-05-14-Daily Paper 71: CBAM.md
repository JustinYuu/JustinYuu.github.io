---
layout: post
title: "Daily Paper 71: CBAM"
description: "Notes"
categories: [CV-Attention]
tags: [Paper]
redirect_from:
  - /2020/05/14/
---

# Daily Paper 71: CBAM: Convolutional Block Attention Module  

## Introduction  

这篇是和BAM一起的姊妹篇，都是由KAIST的韩国人发表的。这篇貌似早一点，发表在ECCV18上，提出了一个卷积块注意力模块CBAM。该模块能够从channel和spatial两个维度来序列性的推断特征图，再将特征图乘以原始的输入特征图，即得到了最终的提炼过的自适应特征图。CBAM是一个轻量化模块，可以应用到多个CNN架构中。作者在ImageNet-1K，MS COCO检测、VOC 2007检测数据集中得到了良好的分类和检测表现，证明了CBAM的有效性和广泛的。  

我看完摘要之后瞬间产生的最大的疑惑就是：这玩意和BAM有啥区别？这玩意和SCA-CNN有啥区别？  

## Method  

没啥好说的，直接看CBAM模块吧，下图是CBAM模块的示意图:  
![71-1](/images/daily paper/71-1.png)  

图示很简单，和BAM的最大区别是这个是串联的。首先得到一个1维的channel特征图，然后得到一个2D的空间特征图，分别使用逐元素相乘来处理原数据。  

然后看看子模块，下图是子模块的示意图:  
![71-2](/images/daily paper/71-2.png)  

对于channel attention的生成方式略有不同，这里采用了maxpool+avgpool并行的方式来代替之前的GAP，生成两个全局语境解释器，然后通过一个shared network生成channel attention map，网络是一个隐藏层的MLP，隐藏层的尺寸降维以降低计算复杂度，降维的比例由reduction ratio控制。通过shared network后，使用逐元素相加来merge起来。使用两种池化方式处理后通往MLP再merge，这种方法应该是一个经验法则，我其实是不太明白AvgPool池化后忽略了什么本应得到但又无法得到而又能通过MaxPool得到的重要信息。  

spatial attention模块也在通道维度上使用了这两种池化方式，然后直接concat起来，生成了一个特征解释器。之后通往一个卷积核尺寸为7×7的卷积层中，生成最终的空间特征图。  

作者比较了两个特征图并联和串联的结果，最后发现串联的结果会好一点(？？？)，所以采用了串联的方式。  

## Experiment  

实验部分就不多放了，消融实验结果略过，放一下在ImageNet-1K上的分类结果:  
![71-3](/images/daily paper/71-3.png)  

以及在MS COCO和VOC 2007上的目标检测结果:  
![71-4](/images/daily paper/71-4.png)  

## Conclusion  

看到这里我其实有点头疼，CBAM和SCA-CNN已经不能用相似来形容了，简直是一个模子刻出来的。。除了SCA-CNN用的是tanh+softmax处理子模块的注意力以外，整体的流程在我看来是完全相同的。。此外，BAM和CBAM的相似程度也相当高，BAM只是把CBAM的串联改成了并联，然后把channel attention中的两个池化方式换成了GAP，spatial attention的池化方式换成了空洞卷积。我觉得这些改进是挺不错的，不管是从想法的新颖度还是最后的表现来看都是相当好的，但是这点东西不至于发三篇文章吧。。至少CBAM和BAM这两篇没必要分成两篇发ECCV和BMVC。如果有看客老爷觉得我有什么没发现的创新点，欢迎在评论里给我答疑解惑一下，感激不尽。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

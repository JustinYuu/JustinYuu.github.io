---
layout: post
title: "Daily Paper 68: SENet"
description: "Notes"
categories: [CV-classic]
tags: [Paper]
redirect_from:
  - /2020/05/08/
---

# Daily Paper 68: Squeeze-and-Excitation Networks  

## Introduction  

SE-Net是2017年最后一届ImageNet分类挑战赛的冠军模型，该篇论文发表在CVPR2018上。之前的MMTM就借鉴了SENet的思想，所以这里把SENet重新回顾一遍。  

SENet比较有意思的一点是他把注意力聚焦在了通道间的关系上，作者提出了一个Squeeze-and-Excitation模块，通过显式的对卷积特征的不同通道间的关联性建模，从而学习到更高质量的表示。这种思想类似于attention和gate，对不同通道的表示进行gating，通过学习的方式来自动获取到每个特征通道的重要程度，然后依照这个重要程度去提升有用的特征并抑制对当前任务用处不大的特征。  

此外，对于新CNN架构的研究一般是复杂的，因为需要寻找一系列超参数的值，但是SENet的插入式架构能够适应当今的很多主流网络，在不需要改变超参数设置的前提下有力的提升网络性能，对于网络的复杂度和计算的负担来说增加的都很少。  

## Method  

![68-1](/images/daily paper/68-1.png)  

上图就是著名的SE模块，对于给定的输入X，由任意转变方式F转化成H×W×C的特征图U后，就可以使用SE模块进行特征的再校准。特征图U首先通过一个squeeze操作变成一个1×1×C的channel descriptor，也就是说把整体的特征图压缩成一个通道的特征图，每一个通道的实数表示该通道内部的全局表示，该操作可以使得整体感受野的信息能够被网络所有的层使用。然后使用excitation操作，也就是一个self-gating机制，将之前的embedding作为输入，产生一系列per-channel的调节权重，这些权重应用到原来的特征图U上，从而生成一个和特征图U尺寸相同的新特征图，新特征图的参数是通过SE模块产生的调节权重调节完生成的。  

作者认为SE模块虽然可以作为基础的block进行堆叠，作为一种新的网络，但是作为传统网络中block的替代部分可能效果会更好一些。虽然SE模块在网络的不同位置进行替代都是相同的，但是在网络的前面几层使用SE模块能够以一种类别无关的方式来激发信息特征，从而强化共享的低层级表示，而在网络的后面几层，使用SE模块就会逐渐变得有针对性，以一种非常类特定的方式来回应不同的输入特征。那么整体看来，SE模块对于网络特征的重校准是贯穿始终的。  

接下来说一下实现的细节。squeeze操作主要是通过一个全局平均池化GAP实现的，把一个H×W的二维图池化为一个实数。作者认为这个实数具有全局的感受野。有意思的是，这种方式能够使得网络任意部分的channel descriptor都能够表征在特征通道上响应的全局分布，即使是在非常靠前的层中也能获得全局的感受野，这是之前的网络都无法做到的。GAP的公式如下:  
![68-2](/images/daily paper/68-2.png)  

Excitation操作的目的是为了完整的捕获通道间的依赖性，从而充分利用squeeze操作整合的特征。我认为excitation像是attention，又像是RNN中的gating。对于excitation的要求有两点，首先比如足够灵活，能够学习到通道间的非线性互动；其次必须学习到一个非互斥的关系，从而保证得到的结果可以强调多个通道，而不是产生一个one-hot的表示。那么作者就使用了一个sigmoid作为激活函数的gating机制来完成，公式如下:  
![68-3](/images/daily paper/68-3.png)  

δ代表ReLU，为了减轻复杂度，作者使用两个FC层组成一个bottleneck来实现这一操作，首先使用一个维度削减层，这里需要一个超参数reduction ratio r，后跟一个ReLU和一个维度增加层。经过了这两个操作后，SEBlock最后的输出通过对U进行rescaling得到，实质上就是一个channel-wise multiplicatcion。公式如下：  
![68-4](/images/daily paper/68-4.png)  

整个SE block可以看做是一个维度间的自注意力，先通过squeeze操作把特征图池化为一个channel descriptor，然后使用excitation操作把channel descriptor映射为一系列通道的权重，再通过逐通道的乘法加权到原来的特征图中。  

SE block可以部署到很多主流网络中，比如VGG, ResNet, Inception等等，部署的流程如下图所示:  
![68-5](/images/daily paper/68-5.png)  

在ResNeXt和ResNet上修改后整体的网络架构图如下图:  
![68-6](/images/daily paper/68-6.png)  

## Experiment  

作者做了挺多实验的，这里也就不列举了，就放一个在ImageNet上的结果吧。  
![68-7](/images/daily paper/68-7.png)  

其他数据集和消融实验的结果就省略了。  

## Conclusion  

SENet的这种对通道间进行gating或者attention的想法取得了很好的结果，与此同时这种思想也可以应用到多模态的融合中，像之前的MMTM就是把不同模态之间的信息当成通道进行squeeze和excitation，从而进行intermediate fusion。我觉得只要能把单模态的全局信息很好的提出来，经过SE操作之后应该能得到不错的fusion结果。  



---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

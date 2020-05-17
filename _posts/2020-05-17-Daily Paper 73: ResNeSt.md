---
layout: post
title: "Daily Paper 73: ResNeSt"
description: "Notes"
categories: [CV-Classic]
tags: [Paper]
redirect_from:
  - /2020/05/17/
---

# Daily Paper 73: ResNeSt: Split-Attention Network  

## Introduction  

这篇文章最近非常火啊，是由Amazon的张航李沐在四月下旬挂在arxiv上的。作者提出了一个ResNet的更新架构ResNeSt，主要使用了一个叫做Split-Attention的模块，使得注意力可以跨越特征图的group。ResNeSt模型在和其他网络具有类似的复杂度的前提下在分类任务的表现超过了其他网络，除此之外在其他下游任务上，比如目标检测、实例分割和语义分割等任务上都取得了性能的改进。  

作者认为虽然ResNet在图像分类任务上表现非常好，但是若把ResNet直接应用到其他下游任务上，可能并不适合，因为其感受野的尺寸有效，并且缺乏跨频道的交互。当今的一些改进方式是进行"network surgery"，也就是在网络架构上进行一些调整，比如使用金字塔模块或者引入远程连接、跨频道的特征图注意力等等，但是这些方法都是在原有的ResNet架构上进行修改的。作者就提出了一个问题，能不能建立一个更为强大的骨架网络，能够产生在多个任务上都更好的通用的特征表示？  

基于此，作者就设计了该网络架构，这也是作者最重要的contribution。作者对ResNet进行了一系列修改，通过使用特征图上的split attention，将特征图沿着通道维度分成了若干个groups和更为细化的subgroups，也就是splits。每一个group的特征表示都是由group内部所有splits的特征表示加权得到的。作者将这个模块称作split-attention block，这种改进仍然保持了原来残差块的简单化和模块化。将若干个split-attention堆叠后，作者就构建了一个类似ResNet的网络，称之为ResNeSt，S代表split。该网络相对ResNet及其变体来说并未额外增加计算复杂度，并能轻松的被其他任务所采用作为主干网络。  

作者的第二个贡献就是建立了大范围的图像分类和其他迁移学习应用的benchmarks。具体来说，作者发现应用ResNeSt的网络能够轻松的在多个任务上达到SOTA，比如图像分类、目标检测、实例分割、语义分割等。  

## Method  

首先放一张ResNeSt的整体图和与之前网络的对比图，如下图:  
![73-1](/images/daily paper/73-1.png)  

SE-Net和SK-Net之前都介绍过，图中的split attention应该指的就是channel-wise attention。这里ResNeSt的架构和之前的这两个网络类似，只不过是将输入的特征图分成了k个基数cardinal，然后在每一个cardinal里面再分为r个split进行卷积操作，每个cardinal里面使用channel-wise attention，也就是这里说的split attention进行聚合，再将k个split attention结果concat起来，通过一个卷积层回复维度，与原始的特征图相结合得到最后的结果。  

据我理解，这个网络应该是把SK-Net和ResNeXt结合在了一起。回想一下ResNeXt，就是把特征图分成不同的group，也就是cardinal，然后再进行卷积操作，这里同样也使用了这一理念，把C个channel分成了k个C/k个channel。然后再将每一个group内的C/k个channel分成C/k/r个split，也就是说原有的特征图通过两次分组被分成了K×R个split。  

![73-2](/images/daily paper/73-2.png)  

上图是一个cardinal group里的split-attention模型图。r个split经过两层卷积后，通过空间维度上的GAP池化为1×1×c的channel vector，然后使用一个channel-wise soft attention进行cardinal group表示的加权融合，特征图每一个的通道的值都使用一个在split上的加权组合产生，第c个通道的值用下列公式计算得到:  
![73-3](/images/daily paper/73-3.png)  

经过split attention处理后，沿着channel维度concat起来得到cardinal group representation V，然后与之前的特征图X加在一起，得到最后的输出Y，这里是借用的残差的思想。如果输入和输出的维度相同，就直接加和就可以，反之则通过一个卷积或者卷积+池化层，这和残差网络的实现细节是一样的，也就不多说了。  

作者放的图是一种cardinality-major view，这是为了方便的表达出整体的逻辑，在实际实现的过程中，使用这种view进行实现会有些复杂。为了方便实现，相同group的split是不一定相邻的，为了方便理解，作者也做出了radix-major view的示意图，如下图所示：  
![73-4](/images/daily paper/73-4.png)  

由图可知，首先作者把整个特征图分成了R×K个group，每一个group都有一个cardinality索引和radix索引，那么相邻radix索引的group放在一起。然后再从不同的split中进行一次总结，把有相同cardinality-index索引，但是radix索引不同的group融合在一起。这个操作等同于把每个cardinal group内的不同splits融合在一起。在这之后，由于第一层级是r个split，所以只需要分成k个group进行两个FC层即可。  

实现的过程中，把1×1的卷积层都整合到一个层中，将3×3卷积用分组卷积来完成，组的数量就是RK，就可以非常简单的实现该架构。  

最后作者提了一下ResNeSt和SE-Net与SK-Net的关系。作者认为当radix=1的时候，ResNeSt的Split-Attention是在每一个cardinal group中做SE操作。而SK-Net引入了两个网络branch中的特征注意力机制，但是SK-Net并未就训练效率和对大型神经网络的缩放上进行优化。作者的网络使用一个cardinal group setting泛化了之前在特征图注意力机制上的工作，并保持了计算的高效性。  

## Experiment  

作者用了相当多的trick，包括label smoothing, auto augmentation, mixup training, large crop size, regularization等等，这里就不一一叙述了，感兴趣的话可以参考一下原文。  

作者在ImageNet上进行了图像分类的测试，结果如下图：  
![73-5](/images/daily paper/73-5.png)  

作者还进行了一系列的迁移学习实验，在目标检测任务上的结果如下图：  
![73-6](/images/daily paper/73-6.png)  

在实例分割上的结果如下：  
![73-7](/images/daily paper/73-7.png)  

在语义分割的结果如下：  
![73-8](/images/daily paper/73-8.png)  

可以看出不管是图像分类还是一系列迁移学习的任务，表现都超过当今的主流方法。  

## Conclusion  

总体来说，作者提出了一个新的Split-Attention模块，通过cardial和group的结合，学习到更为强大的表示，在分类、检测、分割任务上同时达到SOTA，还是非常强的。同时这篇文章介绍了相当多的调参技巧，可以学习一下。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

---
layout: post
title: "Daily Paper 69: SKNet"
description: "Notes"
categories: [CV-classic]
tags: [Paper]
redirect_from:
  - /2020/05/12/
---

# Daily Paper 69: Selective Kernel Network  

## Introduction  

SKNet发表在CVPR2019上，按辈分属于SENet的弟弟，这两种方法一脉相承，非常类似。  

在传统的CNN中，每一层的人工神经元的感受野都是相同尺寸的，而作者的灵感来源于神经科学领域中，视觉皮层的感受野是由刺激调节的，但是在CNN中却并没有类似的研究。作者就提出了一个CNN中的动态选择机制，使得每一个神经元都可以基于输入信息的多尺度进行自适应的感受野调整。作者设计了一个新的unit叫做Selective Kernel，该模块内含多种尺寸的卷积核，然后使用softmax attention根据分支内信息的指导进行结果的融合。不同的分支内的不同attention产生不同大小的融合层神经元有效感受野。多个SK单元的堆叠就组成了新的网络SKNets，该网络在ImageNet和CIFAR数据集上都取得了SOTA的表现。  

作者的SK单元包含三个操作：split，fuse和select。Split操作生成多个不同尺寸的卷积核，从而对应了多个不同感受野尺寸的神经元。Fuse操作从多个路径中结合和整合了信息，从而获得了一个全局的具有表达力的用于权重选择的表示。Select操作根据选中的权重整合了不同尺寸卷积核的特征图。  

## Method  

SE unit整体的流程图如下图所示:  
![69-1](/images/daily paper/69-1.png)  

上面已经提到，整个模块由split,fuse和select组成，上图只用了两个kernel作为示例，但是事实上可以使用多个kernel。下面详细介绍一下这三个部分。  

Split模块的输入是任意的特征图X，首先进行两个变换，使用两个尺寸为3和5的卷积核进行grouped/depthwise卷积，BN和ReLU这一套标准的卷积流程。为了方便运算，对于5×5卷积作者使用的是3×3的dilated卷积，dialation尺寸为2。  
Fuse模块的作用是得到一个紧凑的channel-wise特征。之前也提到过，作者整体的目标是为了使神经元能够自适应的调节他们的感受野尺寸。最为基础的想法是使用门来控制多个带有不同尺度信息的信息流进入下一层。那么首先这个门需要能把所有分支的信息整合一下，这里使用的是逐元素相加。之后作者参考SENet的做法，使用GAP来获得channel-wise statistics，得到1×1×C的channel descriptor，然后使用一个FC层来降维，得到一个紧凑的特征z，FC后接BN和ReLU。最后z的维度为d×1，d最小为32。  

最后是Select模块。使用一个soft attention，通过之前学习到的feature descriptor z，选择不同的信息空间scales。具体来说，使用一个softmax来应用到channel-wise维度上，公式如下:   
![69-2](/images/daily paper/69-2.png)  

最后的特诊图V就通过这些注意力权重a,b,c...来加权得到，权重的总和为1。这里的图示和公式都是以2个branch为单位的，其实可以扩展到很多。  

最后看一下整体的网络架构，如下图所示：  
![69-3](/images/daily paper/69-3.png)  

上图对比了ResNeXt、SENet和SKNet-50的网络架构，其中M代表分出的branch数量，G代表grouped convolution的参数，r代表z的维度缩小比例。该网络的整体架构还是在ResNeXt的基础上的，但是SKNet还可以扩展到其他网络上，比如MobileNet和ShuffleNet中，这些网络的架构就不介绍了，之前都复现过，把3×3的depthwise卷积部分改一下即可，改进方式都大同小异。  

## Experiment  

简单放一个结果吧，在ImageNet上的实验结果如下图:  
![69-4](/images/daily paper/69-4.png)  

其他数据集上的结果和消融实验就略过了。  

## Conclusion  

这篇文章的结构很好懂，特别是读了SENet之后，发现改进的其实很少。总结一下文章的思路吧。随着ResNeXt出现之后，增加cardinality来轻量化卷积核被证明是有效的，而inception使用了平行的多卷积核设计。既然卷积核的计算负担被轻量化了，那么作者就自然而然的想到能不能多使用几个卷积核，在当今计算负担减轻的情况下，能不能用一种可接受的计算复杂度完成表现更好的网络架构更新。那么把卷积核增加之后，相较于inception简单的fuse，这里想到了使用现在很火的注意力进行merge，类似于select and fuse的多模态融合操作。  

本来想看看SKNet能不能用在多模态融合上，结果发现这里的最主要贡献是把一路网络分成多个kernel，然后使用注意力来融合，那么单看融合方面就和使用soft-attention进行融合差不多了，好像没啥更新的方向，反而是和之前SENet共有的Squeeze and Excitation部分倒是可以考虑到多模态融合中，就像之前的MMTM。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

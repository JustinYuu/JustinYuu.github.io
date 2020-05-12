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
之前也提到过，作者整体的目标是为了使神经元能够自适应的调节他们的感受野尺寸。


---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

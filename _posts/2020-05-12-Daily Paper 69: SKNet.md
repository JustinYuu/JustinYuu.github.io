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








---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

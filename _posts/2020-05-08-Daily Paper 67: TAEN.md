---
layout: post
title: "Daily Paper 67: TAEN"
description: "Notes"
categories: [CV-Meta]
tags: [Paper]
redirect_from:
  - /2020/05/07/
---

# Daily Paper 67: TAEN: Temporal Aware Embedding Network for Few-Shot Action Recognition  

## Introduction  

这篇是IBM和Technion在4月底挂在arxiv上的，和之前的TARN只有一字之差，做的也是小样本动作识别的工作。TAEN的全称是Temporal Aware Embedding Network for Few-Shot Action Recognition, 这里比较好玩的是他在两个任务上进行了数据集验证，分别是视频分类和时序动作检测上，这正好是我比较感兴趣的两个任务，该算法同时在这两个任务上实现了STOA。  

作者的idea来源于对于动作识别方式的思考。最近很火的双流网络，比如C3D,I3D，通常都是处理短的视频片段，比如16个连续帧，这样就会损失掉一些含有复杂动作的长期轨迹，因此大家一般使用子动作分解的方法作为补救措施。这些子动作是视频内一个长期动作的短时序片段，而作者将这些子动作看成一系列连续的时序片段，顺序由它的时序顺序来决定。而作者的方法试图使用这些子动作和它们的时序顺序来将动作表征为度量空间内的一个轨迹，最终能够在一个与一系列动作相关的深度特征空间内学习运动的类型，如下图所示:  
![67-1](/images/daily paper/67-1.png)  

这个图画的更像是整体算法训练的流程图，我认为这里最大的创新点在于他们使用了动作的子动作生成embedding，然后根据特征空间内的子动作轨迹的相似度来进行分类。这个表示能够把语义、细化的时序表示和时序在一个粗糙的细粒度程度上同时表达出来。和其他方式不同的地方在于，其他论文学习到的embedding并没有明确的指出是什么类型的embedding，或者说对于embedding和特征空间内到底存了什么东西并没有很明确的阐述，而这篇文章明确的指出学习到的embedding是子动作的时序序列，在特征空间内比较的是轨迹的相似度。  

作者认为他们的contribution有三点：首先提出了一个新的度量学习方式，其次针对这个衡量轨迹相似度的学习方式提出了一个新的loss，最后再小样本动作识别的视频分类和时序动作检测上取得了STOA的效果。我觉得真正的contribution是把动作识别和时序动作检测的子动作检测和iDTF转移到了小样本中，这应该是属于动作识别+时序动作检测与小样本的结合体。  

## Related Works  

这篇的related works还是很有意思的，对于子动作检测部分，作者提到了比较著名的ActionVLAD，将子动作和深度神经网络合在一起应用到动作识别任务中。此外还提到了比较关键的iDTF，即hand crafted dense trajectory feature和基于iDTF的时序动作检测方法Real-time temporal action localization in untrimmed videos by sub-action discovery(BMVC2017)。  

对于小样本动作识别部分我就不多说了，我博客里介绍的那几篇已经是近几年所有的研究了。作者只引用了已发表的两篇文章CMN和TARN，当然这个领域还有没发表的两篇，分别是Daily paper 56中1月份的一篇和Daily Paper54中去年六月份的一篇，不过这两篇和作者的算法也没什么关联。  

对于小样本时序动作检测我认为是一个冷门到不能再冷门的领域。据我所知现阶段有两篇类似的研究，一篇就是作者所唯一引用的CVPR2018的One-shot action localiztaion by learning sequence matching network，这篇文章使用matching network的思想来处理这一问题，另一篇更早，是ACM MM2017的SSAD网络。我觉得这个领域真的是大有前景，因为前面两篇发表在2017和2018年，而这几年时序动作检测已经出了无数种新方法，目标检测领域也多了无数种新的算法可供移植。就算完全使用相同的小样本方式，就更新时序动作检测方式本身我估计也能提高性能挺多。  

## Method  










---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

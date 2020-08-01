---
layout: post
title: "Daily Paper 58: AVSDN"
description: "Notes"
categories: [MMML-Event_Localization]
tags: [Paper]
redirect_from:
  - /2020/04/28/
---

# Daily Paper 58: Dual-Modality Seq2seq Network for Audio-Visual Event Localization  

## Introduction  

这篇是国立台湾大学发在ICASSP2019上的，是对Multimodal Event Localization的一篇改进文章。ICASSP的文章大家都懂，所以我就简要介绍一下算法就好了。  

作者提出了一个叫做Audio-Visual sequence-to-sequence dual network(AVSDN)的网络，使用seq2seq的方法促使模型学到全局和局部的事件信息。该算法能够同时应用到全监督和弱监督的event localization中，并超越SOTA的表现。  

我们都知道所谓的SOTA其实就只有一个，就是前面的那篇AVE-ECCV18，但是作者认为那篇论文的作者只考虑到了视频和音频内部的时序关系，而没有考虑到跨模态的关系，所以作者使用了一个基于seq2seq和自编码器的模型，将每一个时间片段的音频和视觉数据作为输入，使用seq2seq的方式来探究全局和局部的事件信息。此外，作者的模型可以用端对端的形式进行弱监督或是全监督的训练。  

## Method  

整体的流程图如下图所示:  
![58-1](/images/daily paper/58-1.png)  

简单来看，和上一篇的区别在于，第一，不使用attention，分别用两个encoder模块处理视频和音频部分，然后分别使用LSTM进行整合，然后进行特征融合；第二，使用一个decoder进行最后的标签预测，将进入LSTM之前的视频和音频特征和融合后的h<sub>f</sub>, c<sub>f</sub>作为输入，得到最终的frame-wise/video-level标签预测。整体流程看起来比之前的还要简单，下面详细的介绍一下。  

首先是encoder，作者同样是使用了在ImageNet和AudioSet上预训练的CNN作为特征提取网络，这里图片的CNN用的是预训练的ResNet-152，音频用的是VGGish网络，音频时长为1s。接下来分别过一遍LSTM，本来就这么少的篇幅居然又介绍了一大堆LSTM，把公式和几个门都写了一遍，我也是醉了。将最后一个时间步T的hidden和cell state作为音频和视频流的全局表示。说白了，就是CNN+LSTM的双流网络作为encoder。  

Fusion模块我期待了半天，结果居然使用的就是上一篇的DMRN。。。  

Decoder使用的是一个单层的LSTM，将CNN处理后的视频和音频特征作为一部分输出，Fusion后的结果作为一部分输出，使用LSTM来得到最后的标签。  

## Experiment  

对于全监督的模型，AVEL（AVE-ECCV18）的最佳表现是72.7，那篇文章的特征提取器是VGG，作者还算有良心，使用ResNet-151替换VGG之后得到的结果是74.7，而作者的结果是75.4。。。行吧，起码提升了一点。  

对于弱监督，AVEL的表现是66.7，换成ResNet-151后的表现是73.3，作者的模型表现是74.2。消融实验主要探究了fusion的作用，就不放结果了。  

## Conclusion  

我很难想象，一篇这种级别的文章居然可以发表在ICASSP上，这篇文章看起来就像是作者看完AVE-ECCV18那篇文章之后灵光一闪写了半个小时代码搞出来的东西，最离谱的是在修改如此简单，性能提升不到1%的情况下，居然能够成功发表出来，不禁让人感慨ICASSP的50%接受率的确不是吹的。作者搞的这个模型，有力的证明了ResNet151确实比VGG16作为特征提取器的效果要好，至于他这个模型到底有没有用，就要各位见仁见智了。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

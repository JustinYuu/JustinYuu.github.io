---
layout: post
title: "Daily Paper 57: AVE-ECCV18"
description: "Notes"
categories: [MMML-Event_Localization]
tags: [Paper]
redirect_from:
  - /2020/04/28/
---

# Daily Paper 56: Audio-Visual Event Localization in Unconstrained Videos  

## Introduction  

这个月写了一个月的代码，东西没搞出来，倒是开了个新坑：多模态的event localization方向，也看了几篇论文，这里把它整理一下。  

这一篇是Audio-Visual Event Localization方向的开山之作，是由Rochester的几位学者发在ECCV2018上的。Event Localization类似于时序动作检测，是将一段视频中的动作发生的时间段进行定位。作者收集了一个用于event localization的数据集AVE dataset，然后提出了三个任务：supervised audio-visual event localization，weakly-supervised audio-visual event localization，cross-modality localization。作者提出了一个audio-guided的视觉注意力机制，去探究audio-visual之间的联系，使用一个dual multimodal residual network(DMRN)去将两个模态的信息融合在一起，使用一个audio-visual距离学习网络去处理跨模态的定位，并获得了很好的效果。总体来讲作者的工作量还是挺大的。  

作者在提出了这三个任务的同时，也给出了这三个任务的basseline和他们的新方法。对于前两个任务，作者将序列标注问题作为baseline，使用CNN来将音频和视觉输入编码，使用LSTM来捕获时序依赖性，使用FC网络进行最后的预测。基于该baseline，作者提出了audio-guided视觉注意力机制来验证是否音频可以有助于视觉特征的提取，该方法同时证明了发声部位的空间定位也能用该方法得到。作者使用上面到的DMRN进行多模态的特征融合。对于弱监督学习，作者将其看做为一个Multiple Instance Learning(MIL)问题，然后通过增加一个MIL池化层来构建网络架构。对于跨模态定位任务，作者提出了一个audio-visual距离学习网络来测量任意视听对的相关性。  

## Dataset and Problems  

### Dataset  

AVE数据集是作者自己搞的一个数据集，因为这个任务是新的，没有对应的数据集进行训练，因此作者就从AudioSet选了一个子集，包括4143个视频，一共有28个事件类，每一个视频中至少含有一个时长为2s的audio-visual事件。事件类的类别跨度很大，每一个类别至少有60个视频，最多的类别有188个视频。  

### Fully and Weakly-Supervised Event Localization  

event localization的目标是预测每一个视频片段中的事件标签。具体来讲，给定一个视频序列，作者手动将其分为T个非重复片段，每一个片段时长为1s，对于每一个片段都有一个标注的类别y，y的类别包括所有AVE数据集的数量加上一个背景标签。对于监督event localization任务而言，事件的标签是对于每一个片段而言的，即1s一个标签。作者研究了单一模态的定位性能和多模态的定位性能，从而判断多模态的模型到底有没有作用。对于弱监督的event localization而言，作者只对整个视频的事件标注了一下，也就是说不是逐秒标注，而是视频整体标注一个tag，但是算法预测的时候仍然是逐秒预测，这种标注的原因是为了能够减轻模型对于精细标注的依赖性。。我觉得这个说的太牵强了，我感觉应该是为了提高模型的泛化性能，提高其应用能力，以及减轻数据集标注的难度。  

### Cross-Modality Localization  

跨模态定位是另一个不同的任务，该任务的目的是去找到某一个单模态的片段与之对应的另一个模态的片段。这就又分为A2V和V2A，即根据音频来找到视频和根据视频来找到音频两方面。该任务是event-agnostic的，也就是说不管什么样子的视频，不管有无标注的视频，都可以进行跨模态的定位。  

## Methods for Audio-Visual Event Localization  





---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

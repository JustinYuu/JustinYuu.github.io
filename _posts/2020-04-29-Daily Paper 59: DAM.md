---
layout: post
title: "Daily Paper 59: DAM"
description: "Notes"
categories: [MMML-Event_Localization]
tags: [Paper]
redirect_from:
  - /2020/04/29/
---

# Daily Paper 59: Dual Attention Matching for AudioVisual Event Localization  

## Introduction  

这篇是audio-visual event localization最优秀的强化版本，和上一篇的ICASSP同一时期发出来，质量却有天壤之别。这篇paper最大的特色是使用了一个Dual Attention Matching双注意力匹配模块去处理视听事件定位问题。作者的主要思想在于同时捕获整体的高级别事件和局部时序的信息，从而得到更为广泛和准确的结果。这种方式可以同时应用在监督视听定位、跨模态定位等一系列audio-visual event localization任务上，并在AVE数据集上得到了STOA的效果。  

作者回顾了之前的ECCV18和ICASSP19的两篇文章，指出这两篇文章的最大问题在于只聚焦了segment-wise的信息，而没有注意到两个模态之间的global temporal co-occurrences。所谓的global temporal co-occurrences，指的是






---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

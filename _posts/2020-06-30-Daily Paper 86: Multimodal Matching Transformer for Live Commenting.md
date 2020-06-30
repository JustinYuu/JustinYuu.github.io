---
layout: post
title: "Daily Paper 86: Multimodal Matching Transformer for Live Commenting"
description: "Notes"
categories: [MMML-Transformer]
tags: [Paper]
redirect_from:
  - /2020/06/30/
---

# Daily Paper 86: Multimodal Matching Transformer for Live Commenting  

## Introduction  

这篇是哈工大和MSRA于二月底挂在Arxiv上的，使用了一个多模态匹配Transformer用于实时评论任务。实时评论任务我一直没想明白有啥用处，但是我理解它作为评价video-to-text生成水平的一个评价指标。目前该任务一般使用encoder-to-decoder架构，但是作者认为这些方法并没有显式的对视频和评论之间的关系建模，因此很容易去生成一些与视频并无关系但是非常流行的评论。这篇文章作者就力求提升实时评论和视频之间的相关性，这是通过对不同模态之间的跨模态关联建模来完成的，使用transformer框架来实现跨模态互动。作者使用其模型在公开的实时评论数据集上获得了SOTA的表现。  

## Method  

这篇文章先介绍了一下Transformer架构，这里我就不多说了，毕竟这几篇全都是Transformer的改进，已经非常熟悉了。首先对该任务进行一下定义，给定一个视频V和一个时间戳t，自动实时评论系统系统视图去从一个待选集Y中选择一个评论y<sub>\*</sub>，该评论被认为是在时间戳t附近的视频片段中最为相关的评论，考虑的要素包括周围的评论C，视频部分F和音频部分A




---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

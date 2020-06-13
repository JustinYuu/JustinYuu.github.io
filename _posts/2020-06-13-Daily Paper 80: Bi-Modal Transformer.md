---
layout: post
title: "Daily Paper 80: Bi-Modal Transformer"
description: "Notes"
categories: [CV-Attention]
tags: [Paper]
redirect_from:
  - /2020/06/13/
---

# Daily Paper 80: A Better Use of Audio-Visual Cues: Dense Video Captioning with Bi-modal Transformer  

## Introduction  

这篇是坦佩雷理工大学于五月下旬挂在arxiv上的，看格式投的应该是WACV或者BMVC。这篇文章主要使用了一个跨模态的transformer来做Dense Video Captioning任务。作者认为目前的Dense Video Captioning任务的主流方法大多只利用了视觉模态的信息，而对于听觉部分几乎完全忽略了。而作者提出了一个双模态的Transformer，使用seq2seq的方式来处理任意模态的输入。作者发现预训练的双模态encoder-decoder架构可以作为一个简单提案生成模块的特征提取器。该模块的表现在ActivityNet Captions数据集上得到了很好的表现。  

Dense video captioning我认为是比audio-visual event localization更为复杂的任务，它不但需要对每个事件进行定位，还需要产生一句话对齐进行解释。该任务一般是看成seq2seq任务来处理，因为输入的是视频序列，而输出是字幕序列。该任务的数据集是ActivityNet Captions,衡量指标是BLEU@3-4和F1指数。  

## Method  

整体的流程图如下图:  
![80-1](/images/daily paper/80-1.png)  

整个算法由两部分组成：双模态transformer和多头提案生成器。首先用VGGish处理视频的音频流，用I3D处理视频的视觉流，用GloVe处理Caption，然后使用N层的双模态encoder来处理音频和视觉特征，然后使用generated proposals对输入的特征进行修剪，根据预先定义的提案数选出指定数量的提案特征，然后将修剪后的特征再次传入encoder中。decoder关注双模态encoder产生的表示以及先前的caption word，并生成其上下文的内部表示，该内部表示被传递给生成器（右侧）以生成下一个单词。  

双模态表示在双模态decoder的每一层都使用，具体来说，encoder生成的表示传入到decoder后，优化了之前生成的caption





---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

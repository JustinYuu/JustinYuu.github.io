---
layout: post
title: "Daily Paper 83: Bridgeing Text and Video"
description: "Notes"
categories: [MMML-Transformer]
tags: [Paper]
redirect_from:
  - /2020/06/19/
---

# Daily Paper 83: Bridging Text and Video: A Universal Multimodal Transformer for Video-Audio Scene-Aware Dialog  

## Introduction  

这篇是北大和中科院发在AAAI2020 DSTC8 Workshop上的，也是个multimodal transformer的文章，这次的任务是Audio-Visual Scene-Aware Dialog任务，具体来说就是给定一个视频，机器要能够不断的回答提问者的问题从而和他聊下去。。。我不太懂为啥会有这么多奇奇怪怪的任务，反正这个任务是一个多模态任务就够了，那就能用multimodal transformer。AAAI2020针对该任务搞了一个竞赛叫做8<sub>th</sub> Dialog System Technology Challenge(DSTC8)，然后作者的算法在多个衡量指标上都得到了最好的表现。  

首先放一张图，看看啥是AVSD任务:  
![83-1](/images/daily paper/83-1.png)  

看起来和VQA差不多，区别在于每一个dialogue都有音频、视频、视频caption和dialogue summary，以及有10段对话。作者认为当前的主流操作是独立的使用encoder对不同模态的信息独立的编码，然后利用注意力机制来将不同模态的表示fusion起来，也就是主流的N-stream网络。作者认为这种方式并不能学习到最佳的多模态联合表示，因此采取了一个全局的多模态transformer来对不同的模态进行联合编码，并与此同时生成response。该任务输入的是多段序列，输出的也是一段序列，所以可以用encoder-decoder架构来完成。作者的模型基于预训练的GPT2模型调优实现，主要的contribution如下：第一个使用预训练的自然语言生成模型来处理多模态对话生成任务；将多模态特征整合到一个encoder中，并提出了一个multi-task学习方法去学习更好的联合表示并生成信息更丰富的回答；在AVSD数据集上实现了SOTA，遥遥领先了DSTC8-AVSD任务的其他队伍。  

## Method  

### Task Formulation  

首先介绍一下该任务以及一些标注。作者的目标是整合多模态信息，包括视频、音频和对话文本来生成富含信息并且流畅的问题回答。V代表视频，A代表音频，由于summary和caption有诸多相似，作者将这两者concat起来生成一个大的caption C，那么假设该对话有N对问答，那么用U={Q<sub>1</sub>,R<sub>1</sub>,Q<sub>2</sub>,R<sub>2</sub>, ..., Q<sub>N</sub>, R<sub>N</sub>}来表示整个问答文本，Q代表问题，R代表回答，其中R也可以分为m个r<sub>nm</sub>，代表m个单词。  

那么搞定了标注，该任务的目的就是给定Q, V, C和历史dialogue U<sub>＜n</sub>，回答给定的问题Q<sub>n</sub>，给出最优秀的答复R<sub>n</sub>的每个单词的概率，用公式表达如下:  
![83-2](/images/daily paper/83-2.png)  

### Model Overview  

模型示意图如下:  
![83-3](/images/daily paper/83-3.png)  









---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

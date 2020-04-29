---
layout: post
title: "Daily Paper 60: AVIN"
description: "Notes"
categories: [MMML-Event_Localization]
tags: [Paper]
redirect_from:
  - /2020/04/29/
---

# Daily Paper 60: What makes the sound A Dual-Modality interacting network for audio-visual event localization  

## Introduction  

这篇paper是IIT Madras的印度人发在ICASSP2020的，又是一篇改attention的文章，不过这篇比ICASSP2019的那篇强一点，attention虽然是搬过来的，但是作者宣称的性能至少提高了超过1个百分点，还是可喜可贺的。  

这篇的模型叫做Audio-Visual Interacting Network(AVIN)，能够通过模态间和模态内的交互来探究两个模态间的局部和全局的信息。该模型可以应用到WSEL和SEL任务上，貌似没提到在CML任务上的应用。  

## Method  

该算法的流程图如下图所示:  
![60-1](/images/daily paper/60-1.png)  

整体的流程分为六个步骤：特征提取、时序依赖性建模、双模态高级别关联捕获、模态内/模态间互动捕获和特征整合、结果预测。  

特征提取用的预训练CNN，时序依赖性建模用的LSTM，双模态高级别关系捕获用的是Bilinear Pooling双线性池化。这对我来说是一个新东西，作者认为传统的fusion的直接element-wise addition and concatenation大法太过直接，无法捕获到高级别的模态间的联系，所以作者使用一个多模态的bilinear model来获得这种高级别的联系，具体做法就是乘以一个投影矩阵，有z<sub>t</sub> = F<sub>t</sub><sup>v</sup> W<sub>i</sub> F<sub>t</sub><sup>a</sup>，这里W是一个三维的矩阵，参数非常大，所以为了简化计算和训练，作者使用了Multi-modal Factorized Bilinear Pooling，将W分解为两个低秩的矩阵，公式如下图:  
![60-2](/images/daily paper/60-2.png)  

这里使用了power normalization和L2均一化来处理，使其更容易收敛。其中U和V是两个低秩的矩阵，更容易学习到，o指的是Hadamard乘积，q指的是潜在维度，使用Dropout来避免过拟合。



---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

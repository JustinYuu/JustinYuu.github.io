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

这里使用了power normalization和L2均一化来处理，使其更容易收敛。其中U和V是两个低秩的矩阵，更容易学习到，o指的是Hadamard乘积，q指的是潜在维度，使用Dropout来避免过拟合。该factorized bilinear pooling可以使每一个片段的音视频特征都能够有效的融合。  

上面的fusion模块只是把两个模态的信息按照片段各自合在了一起，而作者提出的注意力机制可以将两个模态之间的全局信息考虑在内，所以作者借鉴了Re-id的思想，使用了self and collaborative attention去捕捉模态间和模态内的交互。模态内的交互是通过对整个片段进行平均池化得到的，用一个特征向量表示一整个视频。然后用每一个时间段的向量与平均的特征向量进行点乘，再进行softmax，再与该片段的向量进行元素乘法，得到了音频和视频的encoded intra modality interactions。再将所有的encoded intra modality interactions取平均，继续如法炮制，与每一个时间段的特征向量和其进行点乘，Softmax之后与每一个时间段的特征向量进行元素乘法，得到encoded inter modality interactions。  

之后将这四个interactions和之前的bilinear pooling结果z concat在一起，共同导入到FC+Softmax层中，得到最终的预测结果。使用multi-class交叉熵来训练这个网络。  

## Experiment  

作者的准确率是75.2和69.4，比AVSDN的72.8和66.5高了2-3个百分点。  

## Conclusion  

这篇文章的novelty在于搞了一套新的fusion和attention方法，得到了略高于STOA的表现，算是一篇ICASSP级别的文章吧。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

---
layout: post
title: "Daily Paper 51: Metirc-Based FSL for Video Action Recognition"
description: "Notes"
categories: [CV-Meta]
tags: [Paper]
redirect_from:
  - /2020/03/20/
---

# Daily Paper 51: Metric-Based FSL for Video Action Recognition  

## Introduction  

这篇paper是去年九月份发表的，几乎是小样本动作识别领域最早的几篇文章了。正如作者所说，在小样本图像分类发展的如火如荼的情况下，对于小样本视频的研究，比如动作识别，目标分割等任务的研究还严重不足。这里作者使用了小样本学习中的metric-based方法，并在自制的小样本Kinetics 600上进行了测试。作者发现原型网络和池化的LSTM网络嵌入能够得到最好的效果。  

近些年，陆续有一些小样本的动作识别研究出现，比如使用model-based方法中的memory augmented neural networks(MANN)及其变体去实现小样本动作分类，或者使用metric-based方法中的关系网络来实现。Cao等人提出了一个时序对齐的方法，从而能够更好的对视频中的时序变化建模，他们达到了当时Kinetics 400的STOA表现，达到了85.8%的准确率。  

## Method  

对于元学习的定义就不再赘述了，作者主要是使用了度量学习的方法，整体的结构是一个双流模型，一个流是RGB图像，通过ResNet-18进行处理，另一个流是光流，通过AlexNet进行处理，之后两者再通过LSTM/Avg/3D Conv三种方式整合，再用相加的方式连接两个流。最后通入小样本学习方法中，也就是度量学习，这里使用三种，Matching Network, ProtoNets和learned distance metric(Relation Network)。ResNet和AlexNet的参数均使用在ImageNet上预训练的结果。  

接下来细说一下整合部分。由于通过双流的网络产生的embeddings是时序的，所以需要通过一系列整合方式来产生一个固定长度的embeddings。这里其实有四种策略，第一种就是简单的平均池化，根据时间取平均；第二种是LSTM，使用RNN来捕获时序信息，将LSTM的隐藏层尺寸设为512，再将隐藏层的值对时间步取平均即可；第三种是ConvLSTM，这主要是考虑到使用度量学习方法时，只用一个LSTM可能不太够，因为LSTM需要flat features，所以作者使用了一个带卷积的LSTM来代替原本的LSTM，将线性子层替换成卷积层；第四种是3D卷积，使用3D卷积的方式来捕捉每一个视频的时空特征，由于多了一个维度，所以不需要将时序压缩了。  

## Experiment  

简要说一下结果，作者在Kinetics 600数据集上进行了采样和实验，采样的方式模拟了miniImageNet的采样，具体的预训练方式就不详细写了。所有的实验都是5-way 5-shot。作者除了设置了一个普通的测试集之外，还设置了一个类别都相当相似的challenge set，从而进一步评判模型对于精细标签分类的性能。  

结果表明LSTM+ProtoNet的模式在两个测试集上的表现都是最好的，分别达到了84.2和59.4的分类准确率。此外消融实验证明了光流+RGB两者的表现高于任何一个单独的表现。  

## Conclusion  

这篇文章可以说是相当简单了，他简单的将当今著名的几个度量学习算法稍作修改，应用在了动作识别的小样本学习任务上面，结果还算不错。整体的novelty我觉得是不太够的，文章一共只有五页，也不知道能发到哪里去。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

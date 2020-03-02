---
layout: post
title: "Daily Paper 43: Gaussian Prototypical Networks"
description: "Notes"
categories: [CV-Meta]
tags: [Paper]
redirect_from:
  - /2020/03/02/
---

# Daily Paper 43: Gaussian Prototypical Networks for Few-Shot Learning on Omniglot

## Introduction  

今天的paper是原型网络prototypical network的进化版本，作者提出了一个新的基于原型网络的小样本分类框架，作者将其称之为Gaussian prototypical networks。简单来讲，原型网络学习了一个图片和其embedding向量的映射关系，然后将embeddings聚类进行分类，而作者的网络的encoder的一部分输出被解释成了一个用于估计嵌入点的置信区间，并作为一个高斯协方差矩阵表达出来，接着作者的网络在嵌入空间上构建了一个直接的类内距离标尺，使用独立的数据点的不确定性作为权重。换句话讲，网络将每张图片都映射为一个嵌入向量和一个图片质量的估计值，然后用高斯协方差矩阵来预测、表征一个置信区间。  

作者发现该网络和原型网络有同样多的参数，但是效果却好了很多，在Omniglot数据集上取得了STOA的表现。此外，作者还进行了训练集上的人工下采样，从而使得表现再次提升，进而说明该模型在不太均匀的、噪声比较大的数据集上可能表现会更好一点，这也更接近现实世界中的应用情形。  

## Methods  

对于Encoder，作者使用了一个去掉最后一个FC层的多层的CNN作为编码器，来将图片映射为高维向量，这和原型网络是差不多的，但是具体的细节有一点不同，这里并不是简单的一对一映射，映射的结果是一个映射向量和协方差矩阵的预测部分。  




---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

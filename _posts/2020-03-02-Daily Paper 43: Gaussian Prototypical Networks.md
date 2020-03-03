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

### Encoder  

对于Encoder，作者使用了一个去掉最后一个FC层的多层的CNN作为编码器，来将图片映射为高维向量，这和原型网络是差不多的，但是具体的细节有一点不同，这里并不是简单的一对一映射，映射的结果是一个映射向量和协方差矩阵的预测部分。  

这里协方差矩阵的预测部分作者给出了三种不同的方式，第一种叫做Radius协方差估计，取得的协方差估计值只有一个数字，协方差矩阵是一个元素值完全相同的对角矩阵，这种方式对方向并不敏感，因为一个图片只生成一个值，但是这种方法是最高效的，作者认为这种方法可能在不同的数据集上表现不同，在不太均匀的数据集上可能更复杂的协方差估计值会更好一些。第二种叫做Diagonal协方差估计，这里协方差估计的维度和嵌入空间的维度相同，也就是说在不同维度的值都是不同的，协方差矩阵仍然是一个对角矩阵，不过对角的元素各不相同，这可以使得网络拥有方向独立性。最后一种方法叫做Full协方差估计，顾名思义，就是对于一个数据点对应的所有的协方差矩阵完整的取出来，作者认为这种方法太过复杂，没有必要，因此并未做进一步研究。  

作者的网络架构和prototypical基本相同，由于不想复现这篇论文，所以细节方面就不写太多了，总之输出的结果是1×1×(D+Ds)，Ds就是协方差估计的预测值，根据上面的三种方式不同，具体的维度值也不同。作者使用了两种框架，一个小一个大，区别就是中间的channel不同。作者还使用了四种不同的方法来将协方差矩阵的输出值转换为一个真实的协方差矩阵，方法的不同主要是函数的不同和参数的不同，函数可选择的有softplus和sigmoid。  

### Episodic training  

训练的方式仍然类似，都是选取一定类别的支撑集和查询集，然后计算所有支撑集的prototype，但是距离的计算会有一些区别，具体来讲公式为d<sub>c</sub>²(i) = (x<sub>i</sub> - p<sub>c</sub>)<sup>T</sup>S<sub>c</sub>(x<sub>i</sub>-p<sub>c</sub>)，p<sub>c</sub>是c类的prototype，S<sub>c</sub>是协方差矩阵的转置。现在看起来就比较明了了，最大的区别就是将prototypical network中的距离的平方乘以了协方差矩阵的转置再开方。作者认为多了这个协方差矩阵可以有效的学习到嵌入空间中类内的方向独立的距离。  

### Defining a class  

这里对于prototype的计算也有了一些区别，并不是简单的求平均，而是将协方差矩阵的转置的对角线取值构成的向量s代入进去，公式为p<sub>c</sub> = ∑<sub>i</sub> s<sub>i</sub><sup>c</sup> ○ x<sub>i</sub><sup>c</sup> / ∑<sub>i</sub> s<sub>i</sub><sup>c</sup>，○代表元素各自相乘，除法同理。下面是整体的算法流程：![pic1](images/daily paper/Gaussian prototypical network.png)





---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

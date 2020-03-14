---
layout: post
title: "Daily Paper 46: Optimization as a model for few-shot learning"
description: "Notes"
categories: [CV-Meta]
tags: [Paper]
redirect_from:
  - /2020/03/06/
---

# Daily Paper 46: Optimization as a model for few-shot learning  

## Introduction  

这篇paper是twitter的两个学者发表在ICLR2017上的。我认为严格来讲不算是metric learning方法，反而更像是model based方法。作者用了一个基于LSTM的meta-learner模型去学习特定的优化方式，进而训练另一个用于小样本学习的learner网络。我觉得读到这篇paper，才真的有点元学习的意思了，之前的metric learning感觉都不太接近元学习的方式，这里的两步法更像是当今meta learning的一般步骤的雏形。作者认为他们搞出来的这个meta-learner可以通过特定的子任务学习到短期知识，同时又可以通过所有任务学习到一些共性的长期知识。这里学习的其实就是参数的更新和模型的初始化，也就是说元学习学到的方法就是如何训练子任务网络，读到这里的时候我其实不是很明白为什么参数的初始化和更新由机器设置就能够解决小样本学习的问题了，还是要看作者的进一步阐述。  

## Task Description  

这里的元学习定义和训练方式与之前并无太大差异，对于K-way N-shot而言，就是训练集有K个类，每个类有N个图片，测试集每一类都有一张图片，测试集和训练集组成一个子任务，也叫做一个episode。  

将视角再放大到整个元学习模块中，其实又可以分为三部分，即元学习的训练、验证和测试，元学习训练部分主要用来训练一个学习过程，使得学习过程可以使用episode中的训练集进行训练，在测试集上取得好的效果，也就是使用episode进行训练，学习一个通用的学习过程。元学习验证部分主要用来选择超参数，测试部分用来评价泛化性能，这就和一般的深度学习方式差不多了。也就是说，整个任务最大的区别就是学习的不是特定的分类，而是一种通用的分类方式，表现形式为参数的更新和初始化。  

## Model  

整体的流程可以用一个算法流程图来表示:  

![1-1](/images/daily paper/Optimization-as-a-model.png)  

整体的流程还是相当清晰的，一共进行n个episode，在每一个episode中，都进行T次循环，来进行learner的学习，更新learner的参数，在每一个episode进行完之后，进行meta-learner的更新，总之就是两层更新。  

对于learner的更新，使用的是标准梯度下降法的变体，即θ<sub>t</sub> = θ<sub>t-1</sub> - α<sub>t</sub>▽θ<sub>t-1</sub>L<sub>t</sub>


---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

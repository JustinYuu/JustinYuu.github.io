---
layout: post
title: "CS231n Chapter 4 - Introduction to Neural Networks"
description: "Notes"
categories: [CS231n]
tags: [Python]
redirect_from:
  - /2019/07/31/
---

# CS231n Chapter 4 - Introduction to Neural Networks      

## Introduction  

这一课的讲师是Serena Yeung，这周就进入到了神经网络的具体细节，研究反向传播的机制,并用计算图来表示任意复杂程度的神经网络。  

## Backpropagation  

反向传播实质上就是链式求导，由于我已经学了无数次反向传播了，所以这里就一笔带过求导环节的介绍了，这里把我之前MOOC里的笔记抬上来:[链接](http://justin-yu.me/blog/2019/02/16/Neural-Network-and-Deep-Learning-Chapter-3/)。值得注意的是max()函数其实就是一个分段函数，所以其导数也是分段函数各自求导，即为1和0.    

到目前为止，我们只是学到了如何用链式求导的法则来求得梯度，要得到反向传播后的值还要再用原始值减去梯度的某个倍数。这里Serena在回答学生的问题的时候，在黑板上写了一个公式，即L对x的偏导等于L对所有参数的偏导的加和，我认为这其实是不准确的，错误的地方在于对于同一层的参数，的确是加和，但是对于不同层的参数来说却是乘积，Serena这里可能是想着重解释一些同一层面上的偏导处理。  

同样的，我们可以用向量化的思想来将所有的数字换成向量，所有的操作都是一样的，只不过这里是以向量为单位的。对于这一部分的计算，一般都会需要有大量的向量或者矩阵的乘运算，这使得我们要保证所有的矩阵或者向量的shape要合适，在Machine Learning那门MOOC的课后作业中就存在大量的矩阵的shape对应练习，我的心得是将所有的shape在一张纸上记录下来，以便后续coding的时候方便对应。  

这里的代码图示中将每一个结点都看做一个gate，然后用极度封装的方式来表示反向传播：用for循环反向遍历所有gate，然后用一个MultiplyGate类来封装前向传播和反向传播，从而完成一个前向/反向传播的API。那么在编程作业1中，我们就会实现正向传播和反向传播，包括之前学到的SVM和softmax。  

## Neural Networks  

讲完了反向传播，我们进入到了神经网络的介绍。在之前我们学习的线性分类中，我们只有一个f=Wx的操作，而现在的2层神经网络中，我们拥有1到多个隐藏层，并用多个非线性激活函数来更好的分类。以下一段引用CSDN博主[suredied](https://blog.csdn.net/suredied/article/details/82320317)的一段：  
这里用了一个线性多分类器作为例子，线性多分类器中的系数ω，其可以视为一个模板向量，最终输出的是每幅图像在各个类别的模板向量上的得分。这个模型的一个不足之处是各个类别仅有一个模板向量。比如现在要匹配一匹马，马可能头朝右，可能头朝左，还可能头朝下在吃草。那么一个模板向量就不够用了。那么考虑一个类别对应多个模板向量，然后再添加一层，将一幅图在多个模板向量上的得分综合一下，得到在输出类别上的得分，这样就构成了一个二维神经网络的雏形。但又考虑到线性函数的复合仍为线性函数，因此，在两个线性函数之间添加一个非线性激活函数。  

简单来讲，W1负责分类，W2负责加权，但是我不太明白为什么要给这两个不太重要的线性层搞这么多功能，重点难道不是在非线性激活函数上吗……紧接着又开始了喜闻乐见的神经网络和神经元的对比，事实上这个对比是相当牵强的，Andrew Ng也说了，两者其实没啥关系，只是形式上有相似性，都是从前面接受刺激然后处理信息，处理之后再传到后面，但是功能上并没有什么相似性，事实上神经元的原理到现在也没搞明白。这里课程中也强调了，生物的神经元要比我们的人工神经网络结点要复杂的多，不要轻易的将两者进行对比。  

这里还介绍了多种激活函数，比如RELU和tanh等等，不过只是简单提了一下，在后面的课程中将会详细介绍。之前讲的2-layer Neural Net，其实就是有一层隐藏层的神经网络，而用神经网络的层数来命名神经网络是更加符合神经网络结构的命名。此外，最后一层隐藏层到输出层称作全连接层(fully-connected)，这个名词在后面会频繁的看到。事实上全连接层这个名词是用于CNN网络架构中的，与卷积池化层相并列。用术语来讲，全连接层起到将学到的“分布式特征表示”映射到样本标记空间的作用，简单来讲就是分类器。在我的理解下，我们目前所学的层都还是全连接层，都起到的是分类的作用。  

本周的syllabus提供了一大堆材料，这里全部列出来，分别是[back prop笔记](https://cs231n.github.io/optimization-2/)、Justin Johnson写的[linear back prop实例](http://cs231n.stanford.edu/handouts/linear-backprop.pdf)以及一些选读材料，选读材料的连接我就不一一放了，这里放上cs231n的syllabus链接去自取:[syllabus](http://cs231n.stanford.edu/syllabus.html)，这里我把一个我认为比较重要的可选内容放上来：[derivatives note](http://cs231n.stanford.edu/handouts/derivatives.pdf)，这是justin johnson所写的一系列求偏导的方法，其中向量和标量的求导我们已经学习过，但是对于矩阵的求偏导非常值得一看。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

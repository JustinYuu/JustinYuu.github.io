---
layout: post
title: "Daily Paper 95: Manifold Mixup"
description: "Notes"
categories: [CV-classic]
tags: [Paper]
redirect_from:
  - /2020/08/11/
---

# Daily Paper 95: Manifold Mixup: Better Representations by Interpolating Hidden States  

## Introduction  

这篇是MILA发表的一篇对于mixup的改进论文，通讯作者是两位大牛: FAIR的Lopez-Paz和MILA的Bengio。与mixup对于原数据进行处理不同，这篇文章的主要贡献是对数据的隐表示进行混合，从而利用数据隐含的语义信息作为额外的训练信号，使得神经网络在表示的多个层级上获得更为平滑的决策边界。以该种方法进行处理的神经网络能够以更小的偏差方向学习类表示，从而获得更好的实验结果。  


作者对现有的SOTA网络的隐藏层表示和决策边界进行了研究，发现决策边界一般来说都比较锋利、不够平滑，此外与数据的关联也很紧密，其次作者发现大多数隐藏层表示空间，不管是否在数据流形上，都与高置信预测值相关联。那么基于以上的发现，作者就提出了一个Manifold Mixup方式，通过在训练样本的隐藏层表示上进行线性组合，从而正则化以上问题。那么由于训练数据的隐藏层表示被混合了起来，因此也需要对其one-hot标签进行类似的操作，给混合的例子一个软标签。  

那么为了证明该idea的有效性，作者做了两个实验，来验证manifold mixup方式和其他正则化方式的区别。首先作者在很小的数据集上进行了二维分类任务，结果如下图所示：  
![95-1](/images/daily paper/95-1.png)  

左半部分是未经过manifold mixup的结果，可看到决策边界非常sharp，并且隐藏层表示的分布非常狭窄，而be两张图是经过manifold mixup之后的图像，可以看到决策边界平滑了许多，并且隐藏层表示的低置信度预测值区域更为广阔。最后，从c和f的比较可以看出不管是在第一层还是第三层，经过manifold mixup处理后的表示就越奇异，其信息熵也越小。  

下一张图是其他正则化信心的表示结果:  
![95-2](/images/daily paper/95-2.png)  

可以看到除了mixup之外的正则化方法都无法有效的让隐藏层表示聚焦在每一个类上，也无法提供一个更广的低置信度区域。而至于原始mixup，虽然能够产生更多低置信度的区域，但是无法扩大类别特定的状态分布。因此到这里，基本可以证明manifold mixup流形混合方法能够带来两个好处：类表示被扁平化为最小的变化方向，以及在这些平面表示之间的所有点（在训练过程中最没有用到的和不在数据流形上的所有点）都分配有低置信度预测值。作者用一句话概括了manifold mixup的作用: Manifold Mixup imporves the hidden representations and decision boundaries of neural networks at multiple layers，这里improve对于决策边界来说是smooth，对于隐含表示来说是flatten，此外还利用了深层隐藏层的插值来捕获更高层级的信息，从而提供额外的训练信号。  

那么基于以上的作用，作者归纳Manifold mixup有以下优点：比其他正则化方法更好的泛化性能，能提升测试样本上的对数似然，能提升受到新变形影响的数据预测结果，并能提升模型对于一步式对抗攻击的鲁棒性（这是由于Manifold mixup能够将决策边界一定程度上在某些方向上远离训练样本，因此对抗样本想要越过决策边界就比较困难）。  

## Manifold Mixup  


---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

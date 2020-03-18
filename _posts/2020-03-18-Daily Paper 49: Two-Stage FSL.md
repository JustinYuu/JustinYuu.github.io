---
layout: post
title: "Daily Paper 49: Two-Stage FSL"
description: "Notes"
categories: [CV-Meta]
tags: [Paper]
redirect_from:
  - /2020/03/18/
---

# Daily Paper 49: A Two-Stage Approach to Few-Shot Learning for Image Recognition  

## Introduction  

这篇paper是普渡大学的学者发在TIP上的，也是去年年底的一篇文章，应该也是比较新的。这篇文章的主要思想是采取了两步法来解决小样本的图像分类问题，其主要步骤是：第一步使用relative feature来将图片映射到一个discriminative space中，然后根据每个类的方差不同，使用一个网络来预测每一个类的方差，最后使用Mahalanobis距离来进行分类，生成每一个类别的mean-class representation；第二步使用一个与类别无关的映射将mean-sample representation映射到对应类别的prototype representation，然后进行分类。  

作者认为他们的contribution有以下几点：提出了一个新的用于物体识别的relative-fearture解释器，并建立了一个用于学习类别方差的框架，从而计算和类别prototypes之间的Mahalanobis距离，还提出了一个新的训练流水线，从而学习一个与类别无关的从class-mean representation到calss prototype的转换关系。总结一下，作者所提出的创新其实就是将普遍的metric learning的训练方式多加了一个stage，即图片到prototype不是一步到位的，而是有一个叫做mean-class representation的中间状态，这个中间状态究竟有什么用，还要看后续的解释。  

## Method  

作者认为该方法同时结合了metric learning和meta-learning的元素，这也体现在该方法的two-stage上面。度量学习的stage用来学习绝对和相对的特征集，然后使用Mahalanobis距离来计算测试样例的标签。而使用相对特征的思想来源于迁移学习中的区域自适应算法，区域自适应算法所研究的是将同一类别的标注源区域数据和未标注目标区域数据尽可能的适应起来，从而提高迁移学习的表现。元学习的stage用来学习用于分类的auxiliary knowledge，具体是指从一个sample到对应的类别prototype的转换，整体的算法流程图见下图:  

![1-1](/images/daily paper/48-1.png)  

整个系统在一个相当大的数据集上进行，每一个类别都有大量的数据，从而学习可泛化的知识，在新任务的小样本中，数据量非常小，可以根据之前学习到的泛化数据来实现快速学习。我觉得这个流程更像是迁移学习，整体的训练样本数还是挺多的。但是这种方式有一些问题，新任务的数据可能和之前的预训练数据集的书籍完全不同，这就会出现高维度和高方差的缺点，因此作者提出了relative features，variance estimator和category-agnostic transformation等一系列补充方法。此外由于传统的训练方法无法模拟小样本的训练，所以作者还是采用了小样本学习所一般使用的episodic training策略，这个策略已经很熟悉了，我就不再赘述了。  

### Relative-Feature-Space Representation  

接下来依次介绍几大部分。第一个就是所谓的相关特征relative-feature空间的表示，由于在一个episode中，query和support set加起来的数量也很少，那么对于一个特征空间来说，其维度相对于数据的总量来说显得非常大，这种绝对特征图内部的稀疏性会导致过拟合和泛化性能差。因此作者提出了相对特征空间表示，其维度是和数据的总量性匹配的，也就是说和原始的绝对特征空间相比维度小了很多。我也不明白为啥特征空间太大了就会过拟合了，可能是embedding可嵌入的选择太多了？作者也没有进一步说明。  

该空间的计算是通过计算某一样本与该episode中的所有样本(包括自己)的欧氏距离的平方根得到的，对于nq+ns个样本而言，特征的维度就是nq+ns，这就实现了高维度到低维度的转换。  

### Variance Estimation  

在使用绝对特征空间f<sub>φ</sub>和相对特征空间f<sub>ρ</sub>将支持集和查询集的图片转化为embeddings之后，接下来要进行的工作是用这些特征来分类。由于小样本的任务对于训练集来说是新的，因此作者并不想让模型和任何类别所联系起来，而是想让这个模型对新的类别有良好的泛化性能，所以用于传统分类的卷积网络是不行的，更多情况下是使用类似prototypical network那样的最邻近算法。这里也一样，先计算出了绝对和相对的prototype，然后再用softmax和距离函数来计算出预测的概率分布值。  

距离的指标一般会采用欧氏距离，比如原型网络，还是Bregman散度的问题，我就不多重复了，但是欧氏距离也会有一些问题，比如欧氏距离假定所有的类别都在嵌入空间中有相同的分布，但是由于类别之间的方差不同，因此分类的结果可能会不太好，下图就比较清晰的解释了为什么不同类别之间的方差不同：  
![1-3](/images/daily paper/48-3.png)  
简而言之，由于每一个类别的范围大小不同，所以欧氏距离这种不考虑范围的方法不够准确，因此作者使用了Mahalanobis距离来直接衡量一个数据点x和一个分布D之间的距离，公式如下图所示：  
![1-2](/images/daily paper/48-2.png)  

由于每一个类别都需要有一个独特的协方差矩阵S，所以需要将协方差S看做每个类别的prototype的函数，但是由于其维度是在是太高了，需要很多参数来建模，而且S还需要是正定的，因此需要有很严格的约束。作者使用了一个球装的高斯分布，对于所有的特征维度都有相同的方差。prototype到协方差的函数f可以用神经网络实现。  

此外在relative-feature空间中，variance其实并没有任何物理意义，因此直接采用欧式距离就可以，使用负对数似然作为损失函数。  

### Category-agnostic Transformation  

特征提取模型和方差估计器训练完毕后，需要进行第二stage的训练。这里主要是想要找到一个从mean-sample的表示到一个类别的prototype表示的category-agnostic的转换关系。用人话来说，之前的prototype仅仅是每个样例的平均值，而作者认为由于类别的数量很少，因此仅仅靠这几个样本得出的prototype并不是真正的prototype，还需要一步转换，这里就是要学习这种转换。  

作者认为某个类别的prototype是与这个类别附近的prototype相关的，但是支持集上由于样本的数量所限，其位置并不一定能够表示prototype所在的真正位置，因为有可能刚好在边缘上，那么如果将相邻位置的prototype也加以考虑的话，会缩小真正的prototype可能所在的位置。那么基于此，作者就构造了一个转换函数f<sub>T</sub>，输入的是所有的prototype组成的矩阵P<sub>r</sub>和一个新类的绝对prototype p<sub>r</sub>。  

作者将f<sub>T</sub>分了两部分，f<sub>T1</sub>(p<sub>r</sub>)和f<sub>T2</sub>(p<sub>r</sub>, P<sub>r</sub>)，第一部分是mean-sample表示的贡献，第二部分是base-class prototype P<sub>r</sub>的贡献，但是作者也说了，base-class prototype也来源于p<sub>r</sub>。  

第一部分f<sub>T1</sub>使用的是一个复杂的非线性函数，直接将某一类别的mean-sample表示p<sub>r</sub>转为真正的prototype p'<sub>r</sub>。这里使用了残差的思想，f<sub>T1</sub> = p<sub>r</sub>W<sub>1</sub> + f<sub>T<sub>11</sub></sub>，其中W<sub>1</sub>是一个缩放矩阵，f<sub>T11</sub>是一个复杂的多层神经网络。第二部分是将已有的prototype应用到新的prototype上，这就是应用了相邻的思想，使用欧氏距离进行距离的比较，然后使用一个softmax概率分布来表示所有的prototype和当前新的prototype之间的距离，用p<sub>c</sub>表示。这里使用了一个阈值t<sub>h</sub>，小于t<sub>h</sub>的概率都视为0，这样可以减少一些噪声的影响，使得无关的类别不会对新类别的定位产生影响。最后的公式形式是f<sub>T2</sub> = p<sub>c</sub> P<sub>r</sub> W<sub>2</sub>， W<sub>2</sub>同样是一个缩放矩阵，其目的是为了削减这一元素对整个仿射函数f<sub>T</sub>的影响，毕竟还有之前的f<sub>T1</sub>。  

下图是整体的算法伪代码:  
![1-4](/images/daily paper/48-4.png)  

## Experiment  

作者在四个数据集上测试了结果，分别是已经被蹂躏烂了的Omniglot，目前标准的miniImageNet, Zero-shot的CUB-200和CIFAR-100，具体的实验细节就不写了，总体的表现还可以，在miniImageNet上的5-way 5-shot达到了STOA，5-way 1-shot也是第二位，仅次于CVPR18提出的PPA(参数预测)方法。在CUB-299和CIFAR-100的5-way 1/5-shot上都达到了STOA，我觉得也是因为很多方法没在该数据集上进行实验有关，给到的baseline还是MAML和ProtoNet那些两年前的老东西。  

对于超参数的影响、结果的可视化、消融实验我都在此略过，需要看细节的可以再去参考下原文章。  

## Conclusion  

总之作者用了一个看起来相当复杂的方法来改进了目前的metric learning，有趣的是他改进度量学习的方法却是元学习，此外还结合了迁移学习的区域自适应的理念，算是小样本学习方法的大融合了2333。其实简单来说，他最大的贡献还是在原有的度量学习方法后面加了一个prototype的优化步骤，以及多用了一个特征提取的方式来建立两个不同类型的prototype，再换了一个距离指标，看起来novelty还是足够的，是一篇内容充实、细节详尽的好文章。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

---
layout: post
title: "Daily Paper 03: Siamese for SCD"
description: "Notes"
categories: [SR-SCD]
tags: [Paper]
redirect_from:
  - /2019/10/10/
---

# Daily Paper 03 - Pre-training of Speaker Embeddings for Low-latency Speaker Change Detection in Broadcast News  

## Introduction  

这篇paper是UIUC和IBM Research AI共同发表在ICASSP 2019的，主要研究低延迟SCD的扬声器嵌入物的预训练。从摘要中可以大题看出，作者使用了Siamese network的思想训练了一个神经网络，通过该网络生成嵌入物，从而应用在很多系统中。  

看了几篇论文，对SCD也算有了一定的了解了，所以这里就多写一下。传统的SCD方式就是使用距离计算的方法，使用滑动窗口来提取特征，然后使用某种距离来计算连续窗口的特征；另一种方法是使用模型来预测，用讲话者改变前后的片段和整体来fit这个模型，然后根据模型预测的分数来决定是否发生了变化，比如用BIC或者高斯似然分数，daily paper01就是用的这种方法。  

那么不管怎样，我们都需要用某种方法来提取语音的特征，常用的方法有几种，首先是i-vector，之前也介绍过，i-vector的表现总体来讲很好，但是其缺点是对于短时长的语音片段提取的效果不好，那么大家一般用BIC，高斯散度或者x均值等对短片段聚类，然后再用i—vector处理。但是这种聚类方法只适用于离线处理，对于低延时的处理要求还是无法满足。  

为了解决这个问题，用神经网络训练得出的嵌入物被用来当做一种i-vector的补充，异或是替代品，研究表明用神经网络得出的结果，在MFCC上的BIC指标表现会更好，尤其是短时延场景。这里的神经网络一般都通过Siamese或者triplet loss来处理双输入的对比性loss，同时一般使用LSTM来处理不同长度的输入。此外，为了生成嵌入物，神经网络还已经应用到了端到端的SCD系统中，daily paper1就是一个典型的应用，在这种系统中，预测结果直接通过神经网络的输出得出，而不需要使用距离阈值来进行判断。  

这篇paper的创新点在于，将speaker embeddings的学习和SCD的端到端方法结合到了一起（……），将两个片段（改变前后）的Siamese嵌入物feed到SCD系统的FC层分类器里，将embedding的学习过程本身作为Siamese层的预处理。这里他们用了三个预处理机制：性别分类，对比损失和triplet loss，之后同时尝试了单独训练分类器和同时训练分类器和预处理层。之后作者将这个网络应用到了低延时的系统中，将最大片段时长限制在了2s，最后得到在ASR决策边界下性能得到了大幅提升。此外，他们还对比了不匹配的测试片段时长下i-vector和神经网络embeddings的表现差异，最后评价了他们所做的低延时的SCD系统在ASR片段边界下的预测性能。  

## System  

其实主要的东西在这篇paper的Introduction里面已经说的非常清楚了，下面的内容就再说一下实现的细节。简而言之，他们的网络就是使用了Siamese网络来用Bi-LSTM同时处理两个片段的输入，然后产生两个embedding，将其导入神经网络分类器中得出最后的结论，该系统最终可以判断对话中改变的次数和具体时刻。  

Siamese网络的细节如下：三个Bi-LSTM层后接两个FC层，BLSTM和FC的衔接是通过计算最后一个BLSTM层在该时刻的前向和后向的平均激活函数值，将结果导入FC层实现的，同时根据前面所提到的，用三种损失函数进行预训练，具体如下：  

#### Gender Classification  

首先是使用二分交叉熵损失来训练一个性别分类的网络，这其实就是一种多类别语音分类的二元简化版本，给出的公式也就是最简单的二元交叉熵函数，没什么好说的。  

#### Contrastive Loss  

Siamese的典型损失函数就是LeCun的对比损失，使用对比损失的公式就好了，也没什么好说的。  

#### Triplet Loss  

Triplet loss是在FaceNet里面提出的，CS231n中貌似也有提到过，主要是通过一个anchor和两个相反的样本positive和negative组成，这里就不讲具体细节了，这篇paper作者大讲特讲triplet的基础概念，也没加个引用，同样的放上了triplet loss的公式，也没什么好说的……  我觉得作者列出这三个小标题知识为了凑字数和页面到4页整……这一段实在是太水了，三个损失函数讲了半张纸……  

## Experiments  

实验采用的数据集是BN-144，有144小时的音频。对于每一个speaker，他们采样ns个segment，对于每一个segment进行np个正采样，之后再从剩余的speaker集中采样np个speaker，每一个speaker采样一个，这样就有np个负采样。由于speaker主要是男性，因此大多数speaker对都是男-男对，因此又进行了subsampleing,将同性别对和异性别对的数量变为了相等，最终共有5000000对，进行triplet分组后共有329000个triplet。  

之后作者又把网络的architecture重复了一遍……我越来越觉得作者有水字数的嫌疑了……在对比损失和triplet loss用欧氏距离的时候，他们在第二个FC层后面加了一个L2正则化层从而使得embeddings之间的距离被约束在某一界限内。Siamese层的输入是19维的PLP特征，这里用了相对于MFCC更为高大上鲁棒性更强的PLP，可能是MFCC已经被做过了吧（误），Siamese层后面的分类层采用简单的三层FC层夹杂ReLU,他们的输入是Siamese层的embeddings输出，或者是对照组的i-vectors和PLPs,输出的embeddings是32维的，因此classifier的输入是double32也就是64维的，最后用一个sigmoid结点来输出最终的概率。  

在测试过程中，语音的片段并不像训练过程中一样固定时长为2s，而是采取了两种方法：第一种就是固定的segment时长，即2s，第二种就是模仿实际情况，完全根据segment的时长来，如果时长小于2s，那么就按照原有的时长来，反之则根据边界的位置从头/尾向后/前截取2s的时长作为segment。这也导致了严重的训练集和测试集的mismatched，因为用于ASR测试的平均时长只有1.49s。  

结果显示，如果简单粗暴的把PLP特征放进去正确率也就是52.2%，基本上等同于闭着眼猜，把i-vector放进去有86.6%的正确率，不得不说还是比较顶的，那么采用Siamese网络后，如果用对比损失，最高的正确率有87.5%，用triplet有89.0%，估计作者的内心有点崩溃，搞这么复杂提升了2.4%的准确率。  

那么作者又进一步做了测试，做了一个Pricision-Recall的坐标图，对之前说的两种片段的长度做了比较，结果显示欧氏距离的Triplet loss有着最好的表现，加上Siamese层同时训练比单独训练Classifier的表现更好。同时作者还report了sigmoid默认阈值为0.5的时候，测试数据的F-measures，结果显示BLSTM的Siamese网络进行预训练得到的结果比i-vectors有更好的鲁棒性和更好的整体表现，在训练测试集样本时长mismatched情况下性能提升了50.7%，matched的情况下提升了6.6%。此外，由于sigmoid作为判别函数的时候默认阈值是0.5，作者发现将阈值设置为0.7的时候F-measures表现更好，同时，测试的比较结果不变。  

## Conclusion  

最终再总结一下，这篇paper作者搞了一个孪生网络作为SCD系统的预训练模块，然后将输出作为一个embedding导入到分类网络中，通过测试不同孪生网络损失函数的表现，得出了最佳的性能以及对应的组合方式。同时，作者还对比了传统方法i-vector和他们的方法的对比，测试得出准确率差距不大，但是F1指数差距很大，同时得出在低时延系统中，BLSTM的孪生网络相比i-vector而言还是有更好的鲁棒性，在mismatched的情况下表现更为优异。  


---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

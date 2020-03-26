---
layout: post
title: "Daily Paper 56: Action Relation Network"
description: "Notes"
categories: [CV-Meta]
tags: [Paper]
redirect_from:
  - /2020/03/25/
---

# Daily Paper 56: Few-shot Action Recognition via Improved Attention with Self-supervision  

## Introduction  

这应该是最近最后一篇paper了，看完这一篇，代表当前主流的图像和视频小样本学习的metric-based方法已经阅读完毕，我就可以滚去写代码了。这篇paper是澳大利亚和牛津合作的，第一版于1月份发表，还在under review状态，是比较新的一篇论文。文章的信息量还是很大的，使用了一个改进版本的时空attention，结合自监督学习，进行视频领域的小样本动作识别。该算法在HMDB51, miniMIT和UCF101数据集上得到了不错的效果。  

作者在Introduction就上图了，用来解释他们的idea，他们想通过自监督的方式来进行数据的增强，顺便提高模型的整合不变性，见下图:  
![56-1](/images/daily paper/56-1.png)  

作者首先在特征空间中使用时空注意力机制，去探测帧中与动作有关联的区域，并highlight关键信息、略过无意义的背景信息。接着作者使用外部的数据增强，训练了一个自监督判别器来识别增强的数据，这样可以提供更多训练数据，并增强模型的鲁棒性，减少不确定性。  

作者引入自监督最重要的目的就是使attention map对时空的增强都具有不变性，不管视频怎么变，如何打乱和整合，关键帧依然是关键帧。但是这用简单的注意力机制是无法完成的，还需要加上一个自监督的loss，用一种类似multi-task的思想来解决这个问题。  

总结一下作者的contribution: 第一，提出了一个用于解决动作识别问题的有效新方法，使用时序和空间注意力机制来定位动作和抑制噪声和背景信息；第二，提出了一个时序和空间的自监督小样本学习方法，通过选择、时空jigsaw来提升了模型的鲁棒性和避免过拟合；第三，作者首先提出了将原始和增强的数据通过自监督注意力机制进行对齐的思想，从而产生了模型的整合不变性。我认为这篇paper最大的贡献还是将自监督引入了小样本学习中，至于注意力机制应该是早就有过的。  

## Method  

### Preliminaries  

作者首先计算了一个二阶表示，用来整合3D卷积生成的动作特征。计算的方式挺复杂的，给我看的有点蒙。首先由下图的公式中搞到一阶表示：  
![56-2](/images/daily paper/56-2.png)  

然后又用一个二阶矩阵和一个通过Power Normalization操作G(X)定义的特征图来获得二阶表示，公式如下:  
![56-3](/images/daily paper/56-3.png)  

### Pipelines  

整体的流程图如下图所示:  
![56-4](/images/daily paper/56-4.png)  

作者首先使用了一个4层的3D卷积网络来提取时空特征，之后使用上述的二阶池化来获得忽略了时空位置但是强调了用于关系学习的动作的co-occurrences。由于二阶池化忽略了时空特征，所以是时空不变的，对于数据增强的时序和空间打乱免疫。因此，二阶矩阵能够作为2D关系网络的输入，来获得两个动作特征的关联程度，而避免了时空特性的影响。  
损失函数使用均方误差MSE，整体上还是挺好理解的。  

### Temporal and spatial attentions  

读到这里我就想吐槽一句，作者的叙述顺序真的是太混乱了，他不是按照流程的先后顺序叙述的，更像是一部分一部分的叙述，并且还是先叙述后面再叙述前面，这就会让我读起来很混乱，感觉叙述过程缺乏逻辑性。这一部分介绍的是pipeline中的注意力模块起到的作用。先上图:  
![56-5](/images/daily paper/56-5.png)  

一般的方法是直接提取时序和空间的注意力，获得一个1×T×H×W的注意力结果，但是这种方式计算量太大，所以作者使用了这种类似inception的架构，将整个注意力块解构成时序和空间的注意力分支，然后用下列公式完成计算。  
![56-6](/images/daily paper/56-6.png)  

使用注意力机制在小样本动作识别里是一件很正常的事情，但是这种定位注意力图需要做到对数据增强的不变性，比如空间的旋转和jigsaw，以及时序的打乱，从而保证用于relation learning的关键帧和关键区域能够准确的定位。但是只使用这种attention还是做不到的，因此需要通过一种自监督的方式来解决。这里的idea和昨天的那篇paper的idea是类似的，都是将时序考虑在内，只不过这里是要使时序打乱，而上一篇是尽可能保证时序的不变。我觉得这也是各有利弊吧，像这里这样把时序打乱可以增强更多的数据，但是也会失去时序特性，而上篇paper能够考虑到时序特性，不过这两种改进都比之前的不增强数据也不考虑时序好，相当于一个方面的两种改良吧。  

### Improved self-supervised attention mechanism  

![56-7](/images/daily paper/56-7.png)  

对于时序自监督，作者将帧顺序打乱，使网络学习到动作的不变性。这里作者采用了一个不同的打乱策略，称为时序jigsaw，将若干连续帧组成一个片段，类似TARN的segment，然后打乱的单位是segment而不是frame(从而保证了动作的连贯性？) 对于空间自监督，作者进行了随机旋转和随机jigsaw permutation。  

作者使用了一种auxiliary task的思想来对待自监督模块，训练了一个模式判别器，用来识别增强数据的labels。对于rotation操作，作者定义了一个自监督的目标函数L<sub>ssl</sub>，公式如下图，其中d是自监督判别器，D是d的参数。  
![56-8](/images/daily paper/56-8.png)  

将这个L<sub>rot</sub>和原本的损失函数L结合起来，就构成了一个自监督的小样本pipeline，不过这个目标函数无法优化attention本身的鲁棒性。作者最初的想法是因为attention无法对数据增强鲁棒，那么进一步的想法是，对于相同的视频，对原视频的attention和对增强视频的attention应该是一样的，所以可以用一个对齐函数来对齐他们，公式如下:  
![56-9](/images/daily paper/56-9.png)  

最终的损失函数是L+βL<sub>ssl+γ</sub>L<sub>align</sub>  

## Experiment  

作者在HMDB511, miniMIT, UCF101三个数据集上进行了小样本采样和测试，在HMDB51上rotation自监督取得了1-shot的最好表现，在5-shot上spatial-jigsaw取得了最好表现，在miniMIT上temporal-jigsaw取得了1/5-shot的最佳表现，在UCF101上rotation取得了1/5-shot的最佳表现。  

整体看来准确率还是挺高的，但是我感觉baseline偏弱，只选了3D卷积的ProtoNet, Relation Networks和SoSN(Power Normalizing second-order similarity network for few-shot learning, WACV2019)，所以我觉得起不太到真正比较的效果，虽然超过了所有的baseline，也不能算是STOA。  

## Conclusion  

我认为作者的工作还是很solid的，但是缺点在于叙述方式过于复杂和混乱，以及结果的评价不太有说服力，并缺乏可视化结果，感觉结果有点单薄。此外这些自监督方式只能取其一，我觉得既然是multitask，不如做的彻底一点，把这几种augmentation方式都弄上，不知道结果会不会更好一点。这篇paper还在under review，我觉得IJCAI和MM的可能性比较大，CVPR感觉冲不太上去。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

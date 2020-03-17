---
layout: post
title: "Daily Paper 48: Learning Embedding Adaptation for Few-Shot Learning"
description: "Notes"
categories: [CV-Meta]
tags: [Paper]
redirect_from:
  - /2020/03/17/
---

# Daily Paper 48: Learning Embedding Adaptation for Few-Shot Learning  

## Introduction  

今天这个比较新，是新出炉的CVPR2020的，主要使用了transformer的思想来处理小样本的分类问题。在度量学习方向上的研究，一般都是直接采用某一种embedding的学习方式，用同样的方式处理所有分类任务，从而进行度量和分类。但是作者认为，每一个任务都是不同的，所要学习到的差异性也是不同的，如果对于所有的任务，都用同一种嵌入函数来处理的话，那么就相当于需要从这一种分类方式中处理多类任务，这显然是比较困难的，结果也肯定会没那么好。作者因此想提出一种task-specific，也就是独立于特定任务的embedding方式，对于每一类任务，都学习到最好分辨的embedding，再用度量学习的方式进行分类，作者认为结果会好一些。作者采用的方式就是Transformer，在小样本的分类上得到了STOA的效果。  

## Method  

作者首先介绍了传统的小样本学习方式。作者将使用的数据集称作D<sup>S</sup>。该数据集的目的是为了学习分类器f，一般需要从数据集S中取出多个样本，组成多个M-way N-shot的小样本学习任务，这里所有的类别都是support set中见过的，比如matching network等算法中都采用这种方式，然后尽可能的降低损失，优化算法。所谓的分类器f，作者认为可以分为两个部分，第一个就是embedding function，将图片映射到一个特征空间中，第二个是最邻近分类器，根据特征空间中向量的距离来判断类别，这也是metric learning的标准操作，作者将这种方式称作task-agnostic，因为不管是什么子任务，特征空间都是相同弄的。  

接下来是作者的task-specific方式，这里的重点模块叫做FEAT(Few-Shot Embedding Adaptation Transformer)。作者认为主要思想在于新增了一个adaption step适应步，在这一步中，得到的embedding被transform了，这里的转换是端对端的函数，输入的一般是无须的包或者集合，输出的是优化后的embedding，整个过程是permutation-invariant顺序不变的，下图是整个算法的流程图。  

![1-1](/images/daily paper/LEAFSL-1.png)  

第二部分就是FEAT的内部transformer，transformer其实就是自注意力机制，作者通过自注意力机制来将每一个embedding和自己的语义信息结合在一起。Transformer其实由三部分组成，分别是query, key和value，三者组成一个三元组，这在前面的attention is all you need里面有详细的介绍过，这里就不多介绍了，这里的query是要处理的embedding所在的instance，key是已被处理的embedding对应的instance，具体的操作流程按下图所示：  

![1-2](/images/daily paper/LEAFSL-2.png)  

在FEAT中，Q=K=V=X<sub>train</sub>，每一个都有独立的投影矩阵。此外作者还用了一种叫做FEAT\*的方法，将qeury image也纳入考量范围内，两者的具体区别见下图：  

![1-3](/images/daily paper/LEAFSL-3.png)  

最后，为了促进embedding adaptation transformer的学习，作者在原有的目标函数上面又累加了一个对比度目标函数，从而保证经过adaptation的embeddings和相同类别的邻居类似，和其他类别不同。总目标函数如下：  

![1-4](/images/daily paper/LEAFSL-4.png)  

作者使用了两个主干网络，在度量学习领域广受欢迎的4层ConvNet和一个ResNet，对于这两个骨干网络都进行了预训练，作者发现预训练过的网络在SEEN数据集上的表现非常优秀，很快就可以收敛，因此可以显著的加速训练。  

## Experiment  

作者进行了相当多的实验，包括标准的小样本分类、小样本区域生成、转导小样本学习，以及大规模的生成&小样本图像分类。数据集使用的是miniImageNet和OfficeHome。Baseline使用MatchNet和ProtoNet这两个task agnostic方法，作者发现ProtoNet的表现比MatchNet在各个维度上都好，因此作者自己的模型也使用ProtoNet作为学习embedding的网络。作者还使用了四种不同的方式来进行通用的adaptation操作，分别是BILSTM,BILSTM\*, DEEPSETS和DEEPSET\*，最后的结果如下图：  

![1-5](/images/daily paper/LEAFSL-5.png)  

消融实验的结果就不写了，下面介绍其他任务的结果。  

Cross-Domain FSL指的是SEEN和UNSEEN的数据分布完全不同，虽然类别相同，但是画风可能不太一样，这种难度会增加一些。Transductive FSL指的是所有的query image一次性全部给出，这样可以通过整个UNSEEN类的图片来进行分析。Generalized FSL & Low-shot Learning指的是测试样例来自SEEN和UNSEEN类别之中，我不太明白搞出这么多花里胡哨却又差别不大的任务是要干什么，总之结果见下图：  

![1-6](/images/daily paper/LEAFSL-6.png)  

## Conclusion  

总之作者提出了一套新的小样本学习的度量学习方式，主要还是添加了一个transformer，从而能够学习与任务相关的embedding，在各个任务上都有良好的表现。我个人感觉小样本图片的度量学习领域在17-20年出现了明显的断层，好在CVPR2020又见到了进一步的发展。我认为度量学习虽然不像model agnostic那样更接近于元学习的思想，但是这种方法的思想是可解释性更强的，我觉得还应该有进一步的应用。  

此外，必须要称赞一下，这篇paper提供了相当详尽的supplementart Material，内容长达七页，也提供了原始代码，对于复现应该非常友好，有时间我也会复现一下。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

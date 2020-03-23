---
layout: post
title: "Daily Paper 53: CMN"
description: "Notes"
categories: [CV-Meta]
tags: [Paper]
redirect_from:
  - /2020/03/23/
---

# Daily Paper 53: Compound Memory Networks for Few-shot Video Classification  

## Introduction  

这两天抓紧把这方面的文章看完，其他的学习任务先放一放。这篇paper是发表在ECCV2018上的，也就是著名的CMN算法。CMN算法是小样本动作识别领域一个比较著名的算法，该网络架构使用了一个key-value记忆网络的范式，每一个key memory包含了多个组成键constituent keys，这些组成键在训练的时候合作运行，从而使得CMN获得一个在更大的空间内的最佳表示。作者还搞了一个multi-saliency embedding算法，通过发现多重兴趣的显著性，将各个不同长度的视频序列编码成为相同尺寸的矩阵表示。例如，给定一个汽车拍卖的视频，很多人对车感兴趣，但是很多人又对拍卖感兴趣，该算法就试图捕捉不同的兴趣所带来的显著性。此外，作者还设计了一个在构成键之上的抽象记忆abstract memory，这两者共同构成了一层的结构，从而使得CMN能够更加高效，且能够被缩放的同时保证表示能够处理多重键。  
这篇paper应该是model-based的方法，作者致力于构造一个新的模型，从而解决该问题。我一直觉得这种方法类似于盘古开天辟地，可能也是因为我不太了解这方面的工作吧，总之感觉还是挺难做的，作者的模型主要还是基于key-value memory networks。我也没读过记忆网络那篇论文，那篇是15年的，感觉也有点老了，搜了一下找到了这张图:  
![53-1](/images/daily paper/53-1.png)  

简单来说，就是输入的文本经过Input模块编码成向量，然后将其作为Generalization模块的输入，该模块根据输入的向量对memory进行读写操作，即对记忆进行更新。然后Output模块会根据Question（也会进过Input模块进行编码）对memory的内容进行权重处理，将记忆按照与Question的相关程度进行组合得到输出向量，最终Response模块根据输出向量编码生成一个自然语言的答案出来。（此段文字和上面图片均来此该[知乎回答](https://zhuanlan.zhihu.com/p/29590286)）  

这个记忆网络貌似是来自于NLP领域的，作者基于记忆网络的原因有两点：首先新信息可以写到记忆里，从而赋予模型更好的记忆能力，更够更久的记住更多的信息；其次储存在记忆模块的信息可以在更长的周期里被记住，也可以更好的链接和获取。在训练的过程中，每一个训练周期的信息都逐渐的累积，然后在测试阶段使用。  

主要contribution可以总结为以下三点。首先，作者提出了一个compound memory network的新概念，能够学习到更好的表示。这里的灵感来源于memory work，不过这里由于视频的结果比图片复杂很多，含有的语义信息也很多，因此使用的是二维向量，而不是memory work中的一维向量来表示。其次，作者使用一系列隐藏的显著性解释器作为构成键，操作方式是扩充自注意力机制，整合上一个新设计的可学习的变量，能够自适应的检测一个视频内部的隐藏显著性类型。对于每一个类型，都会学习一个隐藏的显著性解释器，然后stack起来作为CMN的视频表示。最后，作者设计了一个层级记忆力框架，在提高效率的时候也能够保持CMN强大的表示能力。第一层储存堆叠的构成键，第二层是一个抽象记忆层，内含有为了检索和更新构成键所需的读取和写入操作。该抽象记忆层将堆叠的构成键压缩成一个向量并快速的提升训练和测试的效率。与此同时，两个层之间的通讯交流保证了抽象记忆能够保持所有的构成键中学习到的信息。  

## Related Works  

没用的和已知的都不提。作者记忆网络的想法一个是来源于memory network这篇NLP的论文，其实更多的想法我认为是来自于CVPR2017年的一篇key-value memory network实现小样本物体识别的论文，他在related works里面提了一嘴，不过作者认为该篇论文没有使用元学习的方法，所以不太好，我也没看过，他说啥是啥吧。  

作者还使用很大的篇幅介绍了memory-augmented neural networks，同样是小样本的model-based方法，这篇paper对于作者idea的影响应该还是挺大的。由于那篇paper的表现不是很好，我又一直在看metric-based方法，所以没看那篇paper，这里引用一篇CSDN[博客](https://blog.csdn.net/qq_34562093/article/details/86591983)补上。  

## Model  

作者的阐述分为两部分，multi-saliency embedding function的介绍和Compound Memory Network的整体流程。  

### Multi-saliency Embedding Function  

整体的embedding model流程图见下图:  
[53-2](/images/daily paper/53-2.png)  



---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

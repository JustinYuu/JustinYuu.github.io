---
layout: post
title: "Daily Paper 89: Adaptive Transformers"
description: "Notes"
categories: [NLP]
tags: [Paper]
redirect_from:
  - /2020/07/08/
---

# Daily Paper 89: Adaptive Transformers for Learning Multimodal Representations  

## Introduction  

这篇文章发表在ACL2020 Student Research Workshop上，主要是对Transformer的改进。作者认为之前的transformer虽然有很广泛的应用，但是参数太多，需要巨大的计算量。因此作者对transformer进行了扩展，使用一些适应性方法来获得更多的模型可解释性和计算效率。具体来，作者研究了注意力的分散，稀疏和结构化的dropout方法，以帮助了解他们的注意力机制如何扩展到视觉和语言任务，此外作者还证明了这些方法能够使得其更了解网络是如何处理复杂的输入序列和不同模态之间的稀疏偏好的。  

总体来讲，这篇文章的贡献如下：首个即将自适应方法从语言特征扩展到多模态特征上，以研究它们如何对齐以捕获不同模态之间的复杂关系，并研究了对齐这些方法带来的影响，从而通过消融分析了解其兼容性；进行了可解释性分析，以了解这些方法如何增强人类对注意力行为和适应性方法的理解；提供了针对多模态输入序列的最新自适应方法的实验结果。总结一下就是搬了一个叫做自适应的方法过来，然后进行了可解释性分析，并得到了不错的实验结果。  

## Background  

由于使用的方法应该不是作者独创的，所以background这一环节其实就相当于之前文章的method部分了。这里主要对作者使用的backbone网络，自适应注意力跨度、自适应稀疏注意力两种自适应方法，以及一个trick LayerDrop进行了介绍，下面分别详细介绍这几种方法。  

### LXMERT  

LXMERT是一个多模态的transformer，作者使用该网络作为baseline架构。自适应方法能够和任意形式的基于transformer的自注意力机制结合，而LXMERT使用自注意力和跨模态注意力来联合表示输入的图片和文本。具体来讲，它将一个词语级别的句子和物体级别的图像embedding作为输入，encoder由9层的单模态语言encoder、5层的单模态视觉encoder和5层的跨模态encoder组成，从而联合表示特征。作者使用的网络在四个任务上预训练过，分别是Masked Cross Modality LM, Masked Object Prediction, Cross Modality LM, Mask Object Prediction, Cross Modality Matching和Image Question Answering，其中视觉特征的ROI特征使用Faster RCNN提取。  

### Adaptive Attention Span  

接下来介绍自适应方法，第一个方法叫做Adaptive Attention Span，动态的attention span能够使得学习最佳的注意力span这一过程本身可以根据注意力头部确定的上下文大小来收集信息。首先介绍一下究竟什么是attention span。所谓的attention span，就是一个注意力头能够联系到的上下文范围大小。如果所有的头都能联系到全文，那么这几个头除了权重不同之外，学习的逻辑其实并没有什么差别。

这里要手动规定一个每个头的span的上界，从而减少计算复杂度和内存开销。由于不同的头根据任务的不同侧重于不同的上下文，作者显式的证明了不同头的注意力span会根据任务的复杂性不同而有非常明显的差异性。作者使用和Sukhbaatar等人几乎相同的设置，公式如下:  
![89-1](/images/daily paper/89-1.png)  

其中z是模型的参数，使用kaiming normal初始化，m<sub>z</sub>与注意力权重耦合，超参数R帮助控制注意力分布的softness。计算的公式如下，其中K代表key，Q代表Query，P代表position embedding:  
![89-2](/images/daily paper/89-2.png)  

之后再使用softmax求得注意力权重的分布，之后用masking function处理，公式如下：  
![89-3](/images/daily paper/89-3.png)  

其中m<sub>z</sub>跟随模型参数一起学习更新。  

### Adaptive Sparse Attention  

接下来是自适应稀疏注意力。这一部分是借鉴Correia等人的工作，使用一个α entmax来代替传统的softmax。这个α-entmax比较冷门，我查阅了最初的论文，公式如下:  
![89-4](/images/daily paper/89-4.png)  

其中α这个参数比较重要，决定了每个注意力头的表现。如果α＞1，那么权重分布就会随着曲率的变化而从softmax的密集表示变化为稀疏的映射。比如当α=2的时候，就可以得到完全稀疏的映射，α的值在1和2之间震荡，该参数也被视为训练的参数之一，在训练的过程中和其他参数/权重一起联合训练更新。  

### LayerDrop  

Layerdrop是2019年由Fan等人提出的一个用于减少transformer深度的优化方法。该方法使用剪枝的策略来减少transformer中相同的子层，作者使用了其中的Every Other策略，也就是根据一个drop rate来丢弃某些层。假设网络的总层数为N，那么p=1代表着在一个模态中，在所有的层中丢掉一个层，剩余层的数量为N-p。虽然去掉之后网络的参数数量仍然和N层的数量相同，但是所有的操作将会变得和N-p的网络一样，从而有效的减少操作的时间。  

## Experimental  

实验细节部分就不多说了，直接放结果吧，在VQA数据集上的结果如下图:  
![89-5](/images/daily paper/89-5.png)  

## Conclusion  

这篇文章更像是一个比赛模型的说明文，作者并没有提出新的模块，而是将一些2019年对transformer的改进用在了多模态任务上，我认为这些改进都可以被借鉴，提到的LXMERT、Adaptive Sparse Attention、Adaptive Attention Span和LayerDrop都值得一试。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

---
layout: post
title: "Daily Paper 63: EmbraceNet"
description: "Notes"
categories: [MMML-Fusion]
tags: [Paper]
redirect_from:
  - /2020/05/01/
---

# Daily Paper 63: EmbraceNet: A robust deep learning architecture for multimodal classification  

## Introduction  

这篇文章是multimodal fusion的一篇文章，19年4月挂到arxiv的，作者是延世大学的两个韩国人。该文章使用了一个深层神经网络进行了多模态特征的融合，并将这个模型应用在多模态分类任务中，取得了不错的效果。  

这篇文章的模型并不是一个端到端的完整模型，而是一个类似于AVE-ECCV18当中的DMRN的多模态融合算法，因此他可以应用在几乎所有需要多模态融合的模型上，这里作者应用的是多模态分类任务，前几天他们又发了一篇应用在多模态动作识别任务上的论文，也取得了很好的效果。  

这里作者认为的多模态包含很多东西，除了最常见的视频之外，还有手机的重力加速度计、陀螺仪等等，作者认为这些装置都可以表达出一些信息，而在实际应用中，并不是所有的模态的信息都会完整的表达出来，可能一个系统会有N种模态信息输入，但是在某一特定的时刻，可能会有一些模态的信息没有输入，那么一些之前的系统会因为某些模态的信息是缺失的而导致效果并不好。为了解决这个问题，作者提出了一个新的深度学习架构叫做EmbraceNet，该架构的主要组成部分叫做docking layers对接层，顾名思义就是将每一个模态的信息转化为一个适合信息整合的表示，然后使用一个叫做embracement layer拥抱层的模块来将表示用一个概率的形式结合起来。作者的架构对任何网络架构都有很好的兼容性，对于不同模态之间的信息有深度的关联性，并能无缝处理丢失的数据。  

## Related Work  

因为第一次接触multimodal fusion这方面的东西，所以把related work简单的说一下。最为广泛的fusion方法就是early integration和late integration(decision fusion)，这在之前的event localization中也看到过。early fusion就是在LSTM之前进行fusion，即特征提取出来就开始fusion，而late fusion是在LSTM之后，非线性函数之前进行fusion，作者也提到了有些方法会同时使用这两种integration从而同时取得两种方法的好处。  

在early integration方法中，所有模态的数据都在最初的stage上进行concatenate，不管是什么网络和分类器，都是用一个流进去的，一般情况下都是提取多个模态的特征向量然后进行concatenate。这种方法已经常见到人尽皆知，比如PCA, LCA等经典的方法都是用的early integration。这样做的好处是只要把多模态的特征合成单模态的特征，那么所有的单模态的优秀方法都可以使用了。不过这种方法的缺点也是显而易见的，它对于多模态的同步要求极高，不同模态之间要非常完美的对齐才能够统一到单一模态上，所以这在一些需要灵活的同步模型上表现的并不好，比如audio-visual speech recognition,我觉得还可以加上个audio-visual event localization。  

由于early integration有如此明显的缺点，所以late integration更普遍的应用在多模态任务上。最为经典的就是双流模型，每一个模态都单独训练一个分类器，这里其实据我所知会有两种方法，一种是直接在每一个模态上进行预测，将最后的prediction用一个λ结合起来，有的人把这种方式叫做decision fusion，而另一种是在每一个模态上进行绝大部分的特征处理，包括特征提取、时序依赖性处理等等，然后结合起来进行单一模态的prediction，这种对应decision fusion，叫做late fusion。最后的decision会有多种方式来确定，一般来说使用加权方式进行结合比简单的乘法效果要好。late fusion的问题是他并不结合多个模态之间的信息，更像是各干各的，所以可能会丢失一些多模态的关联信息。当然现在的很多方法都是用late fusion+注意力的方式进行算法设计，算是一个折中的好方法。  

还有一些方法，采取更为复杂的方式来调整不同模态之间的贡献度，这是最近比较流行的一种方法，把注意力聚焦在判断当前状态下何种模态的贡献更大，从而动态的调节不同模态之间的协调关系。比如Bharadwaj等人就提出了一个基于语义调节的re-id系统，通过对不同模态的信息的质量进行评估，进而对数据进行优先级定位，从而选择适合的分类器。Goswami等人使用了一个pool adjacent violators算法，将多个分类器结合起来，从而根据不同分类器的置信值对分类器的输出进行校准。Choi和Lee，也就是这两个作者，搞了一个activity recognition模型，通过衡量不同莫泰之间的可靠性来控制每一个模态信息的数量。这种方式能够很有效的提升模型的整体表现。  

最近，CNN和RNN不仅应用在了特征的提取和处理中，也同样应用在了特征的融合中。Ordonez和Roggen将early integration方法和神经网络结合在了一起，将所有的raw sensor channels concatenate在一起，然后导入到一个由CNN和RNN组合而成的深层神经网络中。Costa等人将late integration方法和神经网络结合在一起，把从CNN获取到的decision结合在一起，提升了music genre classification任务的表现。不过我认为这个好像不是CNN的特征fusion，而是把CNN处理后的结果fusion在了一起，好像与这部分没啥关系。与此同时，还出现了新的integration方式: intermediate integration方法，在深度学习的中间部分把特征结合在一起，比如使用bilinear pooling和过几天要介绍的MMTM。这种方法不仅在一个网络里处理了不同domain的模态信息，还考虑到了模态之间的中间级别信息。  

此外作者还提到了数据的丢失问题。这个不是我想看的重点，但是在实际应用中应该会经常碰到。在视频中可能会出现消音的问题，那么这个时候对于null的部分如何处理也是一个需要解决的问题。Ordonez和Roggen使用一个线性插值的方式来处理human activity recognition tasks的数据丢失问题，Ngiam等人在训练audio-visual双模态网络的时候使用的是零填充，Eitel等人使用color and depth images来填充目标检测的噪音。这些方法可能会部分提升算法对于丢失数据的鲁棒性，但是这些简单的问题并不能根本的解决问题。一个很容易想到的方式是使用预训练的自编码器来预测丢失的部分，Jaques就是这么做的，但是如果预测的不太准确的话，结果也会不太好。  

也有一些基于深度学习的模态信息丢失解决方式。比如Srivastava和Salakutdinov等人建立一个深度玻尔兹曼机来学习多模态数据的共享表示，从而能够无视丢失模态的信息，通过Gibbs采样方式来生成多模态共同表示。Gu等人提出了一个包含多融合层的深度学习模型，使用单模态频道来学习modality-specific特征，使用一个双模态联合频道来学习关联的信息。作者认为第一种方式会严格的深度学习架构限制，而第二种方式有模态数量限制，只能2个模态，所以想要提出一些改进。  

## Method  

下图是整体的流程图:  
![63-1](/images/daily paper/63-1.png)  

由上图可以看出，整体的模型由docking layers和embracement layeyr构成，docking layer的数量和参与的模态数量相同，首先通过一个仿射变换处理各个模态的表示x生成z，然后经过一个网络的处理得到d。之后进入embracement layer，首先经过一个变换r得到d'，然后concatenate起来得到最终的向量e，最终的向量e的维度和每一个d都是一样的，然后再通过最后的terminal network得到final decision。  

下面依次介绍两个layer。首先是docking layers，之前提到过EmbraceNet是一个嵌入式的网络模块，它将之前处理不同模态信息的网络的输出作为输入，那么既然可以处理任意形式的输入，第一步就是要把这些混乱的输入整合成为相同的形式，即作者所说的dockable vector。整个docking layers的操作过程其实非常简单，就是进行一个线性变换，然后经过一个非线性层就可以了。

然后是embracement layer。假设一共从docking layers中得到了m个向量，每一个向量含有c个值，那么embracement layer的作用就是将这m个c×1的向量处理为1个c×1的向量，作者将其称为embraced vector。最为简单的操作方式就是逐元素相加，但是这对于部分缺失的数据来说过于致命，所以作者应用了一个基于多项式采样的elaborate fusion技术，也就是图上的操作r。  

定义r<sub>i</sub> = \[r<sub>i</sub><sup>(1)</sup>,  r<sub>i</sub><sup>(2)</sup>, ..., r<sub>i</sub><sup>(m)</sup>]<sup>T</sup> (i=1,2,...,c)是一个来自于多项式分布的向量，即 r<sub>i</sub> ~ Multinomial(1, p)。其中p=\[p<sub>i</sub><sup>(1)</sup>,  p<sub>i</sub><sup>(2)</sup>, ..., p<sub>i</sub><sup>(m)</sup>]<sup>T</sup>, ∑<sub>k</sub>p<sub>k</sub> = 1。这一分布保证只有一个r<sub>i</sub>的元素是等于1的，其他的值都等于0.接下来向量r<sup>(k)</sup> = \[\[r<sub>1</sub><sup>(k)</sup>,  r<sub>2</sub><sup>(k)</sup>, ..., r<sub>c</sub><sup>(k)</sup>]<sup>T</sup>]就被应用到向量d<sup>k</sup>上，即有d'<sup>(k)</sup> = r<sup>(k)</sup>○d<sup>(k)</sup>。其中○代表Hadamard乘积，也就是逐元素相乘。  

最后embracement层的第i个输出向量的组成元素e = \[e1,e2,...,e<sub>c</sub>]<sup>T</sup>就通过下列公式获得:  
![63-2](/images/dailya paper/63-2.png)  

向量e作为terminal network的输出，然后该网络输出最后的预测结果，作为给定的分类任务的结果。之前的过程保证了只有一个模态数据可以对embraced vector的每一个元素起到作用，这其实是一种模态的选择，在每一个e的组成元素中都独立的进行一遍。那么最后的embracement layer的输出就可以通过所有模态的所有数据中获得，因此多模态信息最终得以整合。整个模态的选择过程由p决定，一般来说p=\[1/m, 1/m, ..., 1/m]<sup>T</sup>能够等概率的选取任一模态，但是在训练的过程中可以动态的进行微调，从而获得更好的表现。  

## Benefits  

作者专门开了一个part来介绍该网络架构的优点。简单来说优点可以分为三点：考虑到模态之间的关联性、处理丢失的数据，以及起到正则化的作用。  

模态见的关联性不用过多解释，因为embracement layer的过程就是各个模态的信息整合起来的。而对于丢失信息的处理，EmbraceNet可以通过调整概率p来实现。定义向量u为当前模态是否存在的布尔数组，当该模态的信息存在的时候值为1，否则值为0，那么对于多项式分布s，可以通过下列公式来调整p:  
![63-3](/images/daily paper/63-3.png)  

说白了，就是当当前模态的信息丢失的话，就把选中当前模态的数据作为最后fusion向量数据的概率调整成0，这就保证了不存在的数据永远也不可能成为最终的数据。  

最后是正则化。作者认为这种选择模态的方式满足二项式分布，由因为任选一个元素的二项式分布等同于伯努利分布，因此该操作等同于正则化。  

## Experiment  

作者将该模型和其他fusion方式进行比较，在gas sensor arrays dataset的图示如下图所示：  
![63-4](/images/daily paper/63-4.png)  

在OPPOTUNITY dataset的图示如下图所示:  
![63-5](/images/daily paper/63-5.png)  

在两个数据集的结果如下图:  
![63-6](/images/daily paper/63-6.png)  
![63-7](/images/daily paper/63-7.png)  

作者还做了相当多的实验，包括调参实验，消融实验等等，这里就不一一介绍了。  

## Conclusion  

作者提出了一个新的用于多模态信息融合的深度学习架构EmbraceNet，该架构对于实战中数据缺失的情况非常鲁棒。这篇文章的想法我认为并不复杂，我原以为是使用深度学习的网络来学习融合方法本身，结果是使用单层神经网络来处理不同模态的特征，然后重点训练何时取何者模态。我感觉现在的注意力机制能够达到比这种算法更为优秀的效果，可能是因为该算法在实战中效果比较好吧。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

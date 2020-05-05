---
layout: post
title: "Daily Paper 64: MMTM"
description: "Notes"
categories: [MMML-Fusion]
tags: [Paper]
redirect_from:
  - /2020/05/05/
---

# Daily Paper 64: MMTM: Multimodal Transfer Module for CNN Fusion  

## Introduction  

这篇文章是微软和Georgia Tech在三月底挂在arxiv上的，提出了一个Multimodal Transfer Module的intermediate fusion模型，用于CNN的特征融合。MMTM利用了多个模态之间的信息来对每一个CNN流的channel-wise特征进行重新校准。与其他intermediate fusion方法不同的是，MMTM能够在不同的空间维度卷积层间进行特征融合，并能够在仅改变很小的单模态网络架构前提下进行多模态特征融合。作者在动态手势识别、演讲增强、动作识别任务上都取得了很好的效果。  

为什么大家都使用late fusion呢，作者认为是因为late fusion比较简单粗暴，很好操作，比如如果使用注意力或者池化来fuse每个维度的1维预测值。而intermediate fusion由于需要在中间环节进行维度的匹配和对齐，因此会导致整个过程比较复杂。此外由于late fusion的每一个stream的操作都经过详细的单模态研究，因此可以直接把这些单模态的研究成果搬来用，或者直接使用CNN的预训练权重，而intermediate fusion需要对底层的框架进行彻底的修改，从而导致预训练权重和之前的单模态处理方式都无法直接拿来用，所以对这部分的研究还比较少。  

作者对intermediate fusion的研究就是为了解决上述的弊端，作者的灵感来源于SENet的squeeze and excitation模块，该MMTM模块可以插入到任意late fusion骨架模块中，每一个MMTM有两个基本单元：一个多模态squeeze单元，用于从所有模态中接收特征，生成一个全局的联合表示；一个excitation单元，使用这个联合表示来适应性的强调更为重要的特征，抑制没那么重要的特征。squeeze unit整合了空间的维度，从而使得全局感受野中来自所有维度的信息都能够在全局表示中使用到，也能使得不同空间维度的模态信息可以整合为一个共同的表示。  

虽然这个模态可以自适应的调整维度，能够应用到不同级别的网络中，但是最优的location和模块数量对于特定的应用是不同的。作者设计了不同的MMTM架构，用于特定的不同任务中。作者经过姿势识别、视听演讲增强、动作识别的实验后，总结出了下列经验法则：首先，增加MMTM到intermediate和高等级特征中是有益的，但是对于低等级特征是无效的，作者认为这是因为低等级特征的模态内相对于中等级和高等级特征而言相对较低；其次，就算在姿势估计这种RGB和深度模态都时序对齐的任务中，squeeze操作仍然可以通过提供全局感受野的信息来提高算法表现；最后，通过gating操作的excitation能够比sum操作效果要好，从而证明了作者的emphasis和suppression机制是有效的。  

总结一下，作者主要做了以下贡献：提出了一个MMTM的神经网络来融合CNN模态间的intermediate features，设计了三套应用于三套多模态任务：姿势估计、视听演讲增强和RGB与身体联合的动作识别的网络架构，并得到了超过late fusion方法的表现。  

## Related Work  

首先是late fusion。作者介绍了多种late fusion的方式，包括加权平均、逐元素相加、bilinear乘积、rank minimization、注意力机制等等。然后是SE模块的来源SENet。我之前手撸SENet的时候，完全没有想到过Squeeze and Excitation能够用在多模态的融合上。在SENet中，SE模块首先对卷积得到的特征图进行Squeeze操作，得到channel级的全局特征，然后对全局特征进行Excitation操作，学习各个channel间的关系，也得到不同channel的权重，最后乘以原来的特征图得到最终特征。本质上，SE模块是在channel维度上做attention或者gating操作，这种注意力机制让模型可以更加关注信息量最大的channel特征，而抑制那些不重要的channel特征。另外一点是SE模块是通用的，这意味着其可以嵌入到现有的网络架构中。这样一看SENet的SE模块可以应用在不同的模态当中，把channel换成modality，就是这篇paper了。  

## Method  

这里作者分了两部分介绍，首先介绍MMTM模块，然后介绍在三个下游任务的应用，由于我想看的不是下游任务，而是这个模块是怎么弄的，所以就不介绍下游任务的具体设置了，只把MMTM模块详细讲一下。  

这里作者为了描述简便，举了一个最简单的双流CNN的例子，假设一共只有两个模态，每个模态分别通过CNN1和CNN2进行特征提取和处理。用N和M来表示空间维度，用C和C'来表示CNN1和CNN2的channel。那么MMTM就把特征A和B作为输入，然后学习到一个全局的多模态embedding，使用这个embedding来重新校准输入的特征，具体通过squeeze and excitation操作完成。整体的流程图如下图所示:  
![64-1](/images/daily paper/64-1.png)  

由于单模态卷积层的输出会受到该卷积层感受野大小的限制，并缺乏全局的语义，所以需要Squeeze操作来讲这些输出挤压到一个channel descriptor中，这里沿用SENet的方法，在空间维度上对输入特征使用全局平均池化GAP，这里空间维度可以是任意的，但是经过GAP之后维度会变得相同。  

Squeeze操作结束后就可以开始Multiodal Excitation。所谓的Excitation，其实是一种attention，在SENet中学习的是各个channel之间的权重系数，这里相对应的也就是不同模态之间的权重系数。这里的操作依然和SENet相同，使用sigmoid的gating操作来完成，如下图所示：  
![64-2](/images/daily paper/64-2.png)  

通过以上操作，就可以实现不同modality之间的gating，也就是emphasis和suppression。那么excitation signal E<sub>A</sub>和E<sub>B</sub>是怎么来的呢，首先需要通过之前squeeze操作产生的两个全局向量S<sub>A</sub>和S<sub>B</sub>中得到一个联合表示Z，这是通过把两个全局向量concatenate之后用一个仿射变换完成的，然后对于每一个模态的excitation signal都通过两个独立的FC层预测得到。那么整个过程共需要学习三组参数，生成Z的仿射变换W和b，生成E<sub>A</sub>和E<sub>B</sub>的参数W<sub>A</sub>,b<sub>A</sub>和W<sub>B</sub>,b<sub>B</sub>。  
## Experiment  

作者一共做了三个实验：手势识别、视听演讲增强、人类动作识别，具体的方式我就不放了，总体来说就是在两个stream的某几个conv层中添加MMTM模块，我把动作识别的流程图放在下面，其余的和这个类似:  
![64-3](/images/daily paper/64-3.png)  

实验都取得了很好的效果，由于没有感兴趣的实验，结果就不放了。  

## Conclusion  

SENet发表在CVPR2018，最早是在CVPR2017 Workshop上提出的，获得了ImageNet 2017竞赛分类任务的冠军。这么一个久远的网络，居然能够在2020年的多模态特征融合中大放异彩，一个是说明了Squeeze and Excitation模块的有效，另一个也反映出这篇paper作者丰富的联想能力。因为我也读过SENet这篇文章，也看过多模态融合方面的文章，但是就是没想到把这两个联合起来，所以还是挺佩服作者的。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

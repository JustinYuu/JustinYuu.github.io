---
layout: post
title: "Daily Paper 70: BAM"
description: "Notes"
categories: [CV-Attention]
tags: [Paper]
redirect_from:
  - /2020/05/13/
---

# Daily Paper 70: BAM: Bottleneck Attention Module  

## Introduction  

这篇文章是BMVC2018的Oral，主要研究attention在深度神经网路中的作用。作者提出了一个叫做Bottleneck Attention Module的简单而有效的attention模块。该模块能整合到任何前向卷积神经网络中，沿着两个分离的路径：通道维度和空间维度来推断出一个特征图。作者将BAM模块放在每一个瓶颈层上，也就是特征图下采样的地方。作者的模块建立了一个层级的瓶颈层注意力机制，能够以端到端的形式来与任意前向网络进行联合训练。作者在一系列数据集上将进行了测试，结果显示该模块能够有效的提升若干网络在分类和检测任务上的准确性，说明了BAM模块的有效性和广泛的适用性。  

作者的贡献主要有三：提出了一个简单而有效的注意力模型BAM，能够在不需要精调的前提下整合到任何CNN中；通过广泛的消融实验验证了BAM的设计；并通过多种baseline的网络架构在多个benchmarks上验证了BAM的有效性。在我看来贡献就一句话，提出了一个注意力模块BAM并验证了其有效性。  

## Related Works  

这篇的related works挺有意思。作者认为很多工作都可以归为attention，即使他们没有明确的这么说。作者的related works介绍的也全是attention的东西，我觉得还挺好玩的。首先是cross-modal attention，在现在看来最经典的应该是ViLBERT，但其实在其之前VQA任务中就出现过跨模态的注意力，将从给定的问题中生成的特征图作为queries来检索图像特征，得到与问题有关的特征。  

然后是自注意力，最火的自注意力应该是transformer，但是Residual Attention Networks也使用了一个hourglass模块来生成中间特征的3D特征图，也算是一种自注意力。此外SENet的SE模块也是通道间的自注意力，不过作者认为他们没有考虑到空间的自注意力，BAM是同时学习了空间和维度间的注意力。  

最后是自适应模块，一些之前的工作建立了一些根据输入能够动态调整输出的自适应模块。比如Dynamic Filter Network，Spatial Transformer Network等。作者认为这可以看做是在特征图上的hard attention。Deformable Convolution Network使用了变形卷积来动态的生成池化的offsets，从而保证只有相关的特征能够从卷积中被池化出来。BAM也是一种独立的自适应模块，能够动态的通过注意力机制来抑制或强调特征图。  

这里补一下soft attention和hard attention。Show, Attend and Tell这篇文章把attention分成了soft和hard两种，hard attention每次只覆盖一个很小的位置，对于任意的时间步t，hard attention都只关注一个位置，也就是说是个one-hot向量。那么这个one-hot向量的概率分布怎么求呢，这里就牵扯到用Jensen不等式把隐式的变量s显式表示出来，得到一个下界，再用log作为替换后进行蒙特卡洛模拟，这与强化学习的关系比较大，主要是需要运用RL的方式来解决不可导的问题。总之hard attention有两个特点，无法直接求导和关注局部位置。而soft attention就与之相反，关注全部位置，不是one-hot向量，而是一组不同的权重组成的向量，这由于这时向量是可微的，就可以直接训练了。这么看soft简直全方位优于hard，不过现在也有用hard attention做的工作，还是具体情况具体分析吧。  

## Method  

主要看一下BAM的模块，流程图如下：  
![70-1](/images/daily paper/70-1.png)  

我认为还是挺好理解的，整个架构基本上就是和SENet类似，把channel attention和spatial attention并联，搞了一个联合attention。  

对于channel部分和SENet类似，使用GAP进行池化得到C×1×1的channel向量，然后通到一个两层的MLP中，中间根据reduction ratio r进行降维，最终得到一个C×1×1的channel attention特征图。都是熟悉的操作，就不多说了。  

对于spatial部分需要引入一个dilation value，来决定感受野的大小。首先使用1×1卷积来降维，仍然使用相同的reduction ratio r。然后使用空洞卷积，从而在获得更大的感受野的前提下提高计算的效率。这里使用了两个3×3的空洞卷积，从而高效的利用上下文信息。最后使用一个1×1卷积来降维到1×H×W，最后使用一个BN来scale一下。这里放一下spatial attention的公式:  
![70-3](/images/daily paper/70-3.png)  

最后是把两部分合起来，由于一个维度是C×1×1，一个是1×H×W，那么首先要把两个维度搞成C×H×W的。这里其实有很多结合的方法，比如逐元素相加、相乘，最大值等等，作者取了逐元素相加，这样对于梯度流来说更新最方便。加和之后，通过一个sigmoid函数来得到最后的3D特征图M(F)，元素的范围从0到1。之后3D特征图和最初的特征图F逐元素相乘，然后再加到原有的特征图中。  

整个BAM模块可以嵌入到任意CNN架构中，嵌入的图示如下:  
![70-2](/images/daily paper/70-2.png)  

## Experiments  

作者在CIFAR-10，ImageNet-1K上进行了分类的测试，在VOC 2007和MS COCO上进行了目标检测的测试。分类的结果如下:  
![70-4](/images/daily paper/70-4.png)  

目标检测的结果如下:  
![70-5](/images/daily paper/70-5.png)  

此外还和SENet的结果对比了一下:  
![70-6](/images/daily paper/70-6.png)  

## Conclusion  

作者搞了一个3D的特征图，灵感应该来自于SCA-CNN和SENet，把SENet的维度特征图扩展为维度+空间的3D特征图，同时应用了空洞卷积，从而一定程度上解决了计算复杂度高的问题。SCA-CNN也应用了类似的3D卷积，不过这里把维度特征和空间特征的串联改成了并联，整个流程创新点我认为比较有限，但是结果很好，也能够应用在很多地方，也算是一篇不错的文章。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

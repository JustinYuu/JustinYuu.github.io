---
layout: post
title: "Daily Paper 75: GCNet"
description: "Notes"
categories: [CV-Attention]
tags: [Paper]
redirect_from:
  - /2020/05/19/
---

# Daily Paper 75: GCNet: Non-local Networks Meet Squeeze-Excitation Networks and Beyond  

## Introduction  

这篇文章真是好东西，是大清和港科大几个在MSRA的实习生写的。从名字就可以看出来，这篇文章主要讨论的是Non-local Networks和SENet的对比与结合。昨天刚看完Non-local Networks，当时给我的直观感受就是方法偏暴力，考虑了太多没用的信息，导致增加了太多本不需要的计算复杂度。当然考虑了太多没用的信息这句话只是我的猜测，而这篇文章的作者真的去做了下实验，发现一张图片中不同位置的non-local模块得到的全局信息几乎是一样的。  

这就问题大了。如果真的是这样，那就说明non-local模块所引以为豪的pairwise信息实质上对于任意pair来说都是一样的，那学习到的长距离依赖性到底是否真的有用就值得怀疑了。在这篇paper中，作者利用了这个发现，基于一个与query无关的公式，在准确率和non-local network相当的前提下大幅降低了计算复杂度。简单来说，之前的non-local network将每一个query点都和所有位置的像素点都计算一下关系，但是现在所有的query只需要计算一次就可以了。这可以大大降低计算复杂度。  

那么细细品一下，比如在一个二维图像内，只需要在每一个通道中的H×W个像素中计算一个query和其他点的关系即可，那么这个思想和SE-Net就非常相似，都是用通道维度上的descriptor来处理每一个通道上的二维特征图，只不过对于整合、变换和增强等实现细节略有不同。作者把SE-Net和Non-local Network结合成了一个由三个步骤组成的用于全局语义建模的通用框架。在这个框架之内，作者设计了一个更好的模块，叫做global context block(GC block)。该模块更为轻巧，能够完成以下任务：构建一个语义建模的模块，从而能够把所有位置的特征整合到一起去建立一个全局语义特征；构建一个特征转换模块，去捕捉通道维度的相互依存；构建一个融合模块，将全局的语义特征和原有得到特征结合一下。我怎么看这个模块和SE-Block还是很像。。作者也提到了区别，语义建模和融合模块和NL block相同，分别是全局注意力池化和相加，而transform步骤和SE block相同，使用的是两层的bottleneck，算是两者的结合吧。总之作者应用该模块去建立了一个全局语义网络GCNet，并得到了比NLNet和SENet都优秀的表现。  

## Analysis on Non-local Networks  

如果说前面Abstract和Introduction还留了点面子的话，这部分完全就是硬碰了。

作者用了较大的篇幅来revisiting，由于昨天刚刚写完，这里就不重复了。总的来说non-local block可以被看成一个全局的语义建模模块，它整合了每个query位置的query-specific全局语义特征，而由于特征图是在每个query位置上进行计算的，所以时间和空间复杂度都是N<sub>p</sub><sup>3</sup>。  

接下来是对于该模块的分析。作者首先对不同query位置的特征图可视化，这里可视化选择的是最常用的embedded gaussain版本在目标检测/分割任务上的表现，结果如下图。  
![75-2](/images/daily paper/75-2.png)  

从图中可以看出，相同图片中不同query位置的特征图几乎是相同的。作者紧接着进行了不同query位置的全局语义距离的分析。作者使用余弦相似度和Jesnsen-Shannon散度逐点进行距离比较，加和后取平均值，结果如下:  
![75-3](/images/daily paper/75-3.png)  

很明显可以看出，距离都很小，这说明不同位置的全局语义基本上是相同的，从可视化中得到的猜测基本上被证实了。  

## Method  

那么接下来的改进就很容易想到了，既然多个query没用，那干脆用一个query就完事了。下图是Embedded Gaussian NL模块和其简化版本。  
![75-1](/images/daily paper/75-1.png)  

为了进一步简化计算复杂度，作者引用了分配率，把Wv移到特征池化前面，公式如下：  
![75-5](/images/daily paper/75-5.png)  

那么操作后的图示如下图的b所示:  
![75-4](/images/daily paper/75-4.png)  

从上图的图b可以看出，简化版本的NLblock可以被抽象成三个步骤：全局注意力池化(通过一个1×1的卷积Wk和softmax函数来获得注意力权重，然后进行注意力池化，从而得到全局的语义特征)，通过1×1卷积Wv得到的特征转换，以及特征整合(使用加法融合法)，作者将这三步用一个公式来表示:  

![75-6](/images/daily paper/75-6.png)  

括号最里面的部分代表语义建模模块，δ函数代表特征转换，F代表fusion函数。  

有趣的是，SE-Net中重要的SE block和简化版本的NL block极为相似，从上面的图c中可以看出，SE block由一个GAP层、一个bottleneck转换层和一个rescaling函数作为fusion模块组成。那么作者就把SE block和NL block结合了一下，提出了一个新的全局语义建模框架GC block，也就是上图的d部分。  

作者认为GC block能够同时获得NL block的长距离依赖性和SE block的低计算复杂度。具体方式是使用bottleneck的transform模块来代替1×1卷积。由于bottleneck有两层，所以提升了优化和训练的难度，所以作者在bottleneck的两层中间，ReLU之前加了一个layer normalization层，起到正则化的作用，方便优化。  

GC block的整个运算过程可用下列公式表示：  
![75-7](/images/daily paper/75-7.png)  

## Experiment  

作者在COCO上的目标检测/分割、ImageNet上的图像分类和Kinetics上的动作识别三个任务上进行了性能测试，在目标检测/分割上的表现如下图：  
![75-8](/images/daily paper/75-8.png)  

在图像分类上的表现如下图:  
![75-9](/images/daily paper/75-9.png)  

在动作识别上的表现如下图:  
![75-10](/images/daily paper/75-10.png)  

## Conclusion  

总结一下，这篇paper是SE-Net和NL-Net的结合体。这篇文章的本意是对Non-local network进行复杂度降维，然后结合了SE-Net上的一些trick，构成了一个新的结合模型GCNet。我觉得这篇文章的创新度做的一般，但是解决的问题比较有意义，也算是一篇不错的文章吧。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

---
layout: post
title: "Daily Paper 96: Visual Transformer"
description: "Notes"
categories: [CV-Vision-Transformer]
tags: [Paper]
redirect_from:
  - /2021/03/30/
---

# Daily Paper 96: Visual Transformer: Token-based Image Representation and Processing for Computer Vision  

## Foreword  

本来我是不想再详细记录看的paper的，但是最近transformer in vision的发展速度快的超乎了我的想象。几乎每天起床后刷一下arxiv，都能刷到很惊人的变革性成果，与transformer相关的论文就像浇了大粪的庄稼，一夜之间齐刷刷的出现了。我意识到自己可能处在CV领域巨大的变革之中，所以想要记录一下transformer席卷CV的过程。  

提到transformer，就不得不比较一下在CV领域得到广泛应用的卷积。卷积和自注意力最大的应用区别是局部和全局的关系，这也是由于卷积所具有的inductive bias和translation equivalent所决定的。由于卷积具有天生的2D维度上的归纳偏置，那么卷积会通过大小一定的kernel天生的将附近的像素点考虑在内，这就使得卷积更注重局部的信息。而transformer刚好相反，由于每一个token之间的距离都被缩减至1，因此tranformer以及其本质的自注意力机制更倾向于从全局的角度捕获信息。  

那又从之前的研究中，我们可以得知，self-attention和convolution在一定条件下可以相互转化。从之前Ramachandran在CVPR2019的文章中，我们已经知道transformer在强先验假设下能够得到图灵完备，而从之前Jaggi在ICLR2020的文章中，我们又可以知道transformer在采用特定位置编码的条件下可以达到在处理图像的原理上和CNN完全相同。那么从理论上，之前的工作已经证明了transformer能够替代CNN，即ICLR2020那篇文章一出现，就预示着transformer在理论上其实可以应用在CV领域的所有任务上。  

从transformer席卷CV界这一现象，我也逐渐对AI学术界的规律有点了解了。其实researchers可以分成几个评级，tier 1的researchers能够提出新的范式，这通常需要理论上的一些证明，比如之前Ramachandran和Jaggi对于transformer和convolution关系的证明，tier 2的researchers虽然无法从理论上进行证明，或者说虽然压根看不懂证明，但是能够捕捉到这些范式的证明背后所蕴藏的价值，以视觉transformer为例，既然transformer能够替代CNN，那在CV界的很多任务是不是也能用transformer来实现呢？于是就有了这一大堆视觉transformer在各大领域的应用文。tier 3的researchers们就稍差一些，可能并不知道有这种范式的证明，只知道现在视觉transformer挺火，那我也挑个任务搞一搞试试。其实每个时代都有不同的行之有效的范式可供使用，如果能够抓住最前沿的范式，并进而想到行之有效的应用方式，tier 1可能达不到，但是达到tier 2水几篇顶会应该是没问题的。以后我的博客也会尽量少记录这些应用文，而是会对ICLR和NeuIPS上一些新的paradigm做一些记录。  

## Introduction  

Visual Transformer其实不是纯粹的transformer架构，而是transformer架构和convolution操作的结合。在作者的introduction部分中，作者提出了3个convolution存在的问题：  

1.Not all pixels are created equal.这句反人权宣言的句子想要说明的是在图像分类的过程中，并不是每一个像素的重要性都是相同的。比如物体的像素就比背景像素重要，而convolution这一操作自然无法分辨哪个像素更重要，所以它会平等的处理所有的像素，从而导致出力不讨好的现象，开销不小，但是效果又不是很好。  
2.Not all images have all concepts. 对于一些low-level的特征，使用特定的卷积核去处理是没问题的，因为每一个图片都会有这些low-level的信息，比如边角和边缘等等。但是对于high-level的信息，比如对狗、猫、耳朵等特定形状的识别，就不需要设置特定的卷积核了，因为并不是每一个图片里都会有狗。所以这些大量的high-level的卷积核可能在识别某些物体时被训练出来，但是在很大一部分时间内(看不到这些物体时)发挥不出作用，造成了浪费。  
3.Convolutions struggle to relate spatially-distant concepts. 卷积无法获取长距离依赖性，这个是老生常谈了，就不再赘叙。  

那么基于以上三个问题，作者就提出了Vision Transformer，如下图所示。  
![96-1](/images/daily paper/96-1.png)  

这个网络通过卷积的操作来处理low-level的特征，这是由于如上所述，low-level的特征在所有的图像中都是需要的，而对于high-level的特征图，作者将其编码成了一系列语义token的集合，然后将其放入transformer中，用来捕获token之间的交互。为什么transformer可以解决上述三个问题呢，作者的解释是：1.自注意力可以动态的判断哪一区域是更重要的;2.将high-level的语义编码成为一些与图片相关的tokens，从而代替了对所有图像的所有概念建模;3.通过自注意力对全局建模，获取长程依赖性。从逻辑上来说，给我的观感解释的很清楚，整个paradigm也是自洽、合理的。  

## Method  

我始终抱着这样一个观点：在所有的vision transformer应用文中，transformer都应是一个工具，而不应是最大的贡献点。这是因为transformer能替代CNN也不是应用文里提出的，而是别人早已证明的。那么一个应用文之所以能够发表，除了在数据集上的点高之外，一定要提出为了能够将transformer应用在这一任务上所做的适配性修改。Visual Transformer应用的任务是最为基础的任务，也是大家的关注度最高的任务：图像分类。而我认为他所做的最有意义的修改，就是对high-level语义的token化，以及将token的反向project。  

如上图所示，Visual Transformer的transformer模块包含三部分，首先将像素分组为不同token，每个token代表不同的语义概念，然后用transformer处理token，最后将这些token投影到原有的特征图中进行下一轮处理。其中第一步和第三步应该就是本文最大的创新点了，以下详细介绍。  

### Tokenizer  

作者的思想其实和图网络/卷积有些不同。在图网络和卷积中，会使用尽可能多的卷积核和图网络结点来表示尽可能多的概念，而在这篇文章的工作中，作者使用少量的visual tokens来表示一个图片中出现的概念，每个图像的tokens都是不同的。以下用X来代表特征图，T来代表token，一共有L个token，这里L为16。  

作者介绍了两种tokenizer，第一种叫做filter-based tokenizer。为了将X映射到T，作者首先将X分为L组，L即为T的个数，也就是维度，这是使用point-wise convolution(类似于1×1卷积)来实现的。然后在每一个group，都在空间维度上对像素进行池化，得到一个token，公式如下：  
![96-2](/images/daily paper/96-2.png)  

这其实是一个空间注意力机制，实现和理解起来也很简单。但是由于很多高阶语义是稀疏的，一次性生成所有图片的所有token其实会导致W_A的维度变得很大，因此W_A更像是一个卷积核，会导致一些计算上的冗余。  

那么基于上述问题，作者就提出了进阶版的tokenizer，叫做recurrent tokenizer。每一层的权重都会基于上一层的token值产生，使得前面层产生的token可以来指导后续层的token产出过程，从而构成一个recurrent，如下图所示:  
![96-3](/images/daily paper/96-3.png)  

这个intuition也很好理解。如果用一个相同的卷积核去生成L个token，那么一个卷积核就得负责理解L个不同的语义。而如果把这L个并列的token转化成层叠的token，每生产一个token，都用之前的token校准一下当前的卷积核，从而用不同的权重来生成不同的token，就能提高有限大的卷积核的利用率，从而提高最终的结果表现。我个人认为这有点dynamic convolution的意思。  

### Transformer  

这个没啥好讲的，就是把16个token扔到transformer计算attention，注意这里没提postional encoding的事情，我觉得是一个缺陷所在。  

### Projector  

目前为止得到了16个token，但是我们还得到特征图，以应用到下游的分类任务上，所以还需要用某种方式把这些token变回去，恢复成特征图。这里采用了一个残差的方式把投影结果和原特征加在一起，如下图所示:  
![96-4](/images/daily paper/96-4.png)  

## Experiment  

作者应用的是分类任务，所以在图像分类和语义分割上进行了实验。对于图像分类，可以直接使用，而对于语义分割，作者将其和特征金字塔网络FPN结合，用Visual Transformer来从金字塔网络的所有特征图中生成token进行处理，再重投影到金字塔上。  

结果就不放了，总之是行之有效的。  

## Conclusion  

简单来说，这篇文章最大的贡献，我认为是在思想上提出了将transformer应用到传统vision任务是可行的，并在实验上屠榜来证明了这一点。此外，在结构上tokenizer应该是最大的贡献，避免了直接将特征放到transformer造成的维度爆炸。这篇文章虽然实现起来比较简单，但是不论是从理论的阐述，结构的设计，还是实验的结果来看都非常优秀，是一篇不可多得的好文章。但是此文章也有一定的局限性，比如没有提到positional encoding,没有探究纯transformer架构的表现等等。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

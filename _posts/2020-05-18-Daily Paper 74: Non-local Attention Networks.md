---
layout: post
title: "Daily Paper 74: Non-local Attention Networks"
description: "Notes"
categories: [CV-Attention]
tags: [Paper]
redirect_from:
  - /2020/05/18/
---

# Daily Paper 74: Non-local Attention Networks  

## Introduction  

这篇也是Attention的文章，发表在2018CVPR上，作者是大名鼎鼎的何凯明大大。这篇文章通过Non-local的方法扩大传统CNN，RNN方法的感知域，使每一点的输出受所有像素点的影响。该模型结构简单，效果提升显著，且可以方便的嵌入到现有网络中。该网络在视频分类和静态的图片分类中都取得了良好的表现。  

作者使用的操作叫做non-local operation，该方法是一个高效、简洁而又泛化性强的用于捕获深度神经网络长范围依赖性的操作，而作者提出的non-local operation是计算机视觉中传统的non-local mean operation的变体。直观的来说，一个non-local操作计算了特征图中一个位置和所有位置特征的加权和，位置可以是时序的、空间的，也可以是时空的，这就代表该操作可以应用在图像、序列和视频问题上。该操作有几大优点：首先相对于RNN和CNN的操作来说，该操作通过及计算任意两点(不管有多远)之间的互动捕获了长范围内的依赖性；其次non-local操作非常高效，可以在层数较小的前提下达到最好的表现；最后，non-local操作保持列输入尺寸的多变性，也可以和其他操作，如卷积等等轻松的结合。  

作者举了一个在视频分类中的应用。在视频中，long-range交互广泛的存在于不同时间点和不同的空间位置上，那么在一个单独的non-local块中，可以直接用前向传播的方式来直接捕获这些时空的依赖性。使用由若干个non-local块组成的non-local网络，就可以在视频分类任务上达到超过当今2D分类网络和3D分类网络的结果，并能获得比3DD网络更小的计算复杂度。  

## Method  

首先解释一下non-local究竟是个什么玩意。所谓的non-local，就是指通过一个算法，能够计算到一张图片中所有像素的加权平均值。作者定义深度神经网络中的non-local操作如下:  

![74-1](/images/daily paper/74-1.png)  

其中i是输出位置的索引，j是所有可能位置的元组索引，x是输入的信号，y是输出的信号，其尺寸和x相同。成对的函数f计算了i和所有的j之间的关系标量，而一元函数g计算了在位置j上的输入信号的表示。结果最后被一个因数C(x)归一化。  

这里的操作就和卷积和递归形成了鲜明的对比。众所周知，卷积需要靠卷积核进行操作，而卷积核的尺寸都是比较小的，那么单个卷积操作势必要在一个局部地区进行。而recurrent操作一般是根据当前和最近的时间段来考虑的，因此也不是全局的信息。此外non-local网络和FC层也不太一样，在上述公式中，计算的结果是基于不同位置之间的关系产生的，而FC层使用了学习过的权重，换言之，在FC层中，i位置和j位置的x之间的关系并不是一个输入数据的函数，而non-local是。  

non-local操作非常灵活，可以和convoluntion/recurrent层一起使用，也可以在深度神经网络的前部使用，这就使得作者可以构建一个同时结合non-local和local information的架构。  

之后介绍一下f和g的细节。作者提出了几个版本，不过作者也做实验证明了具体选取哪个版本好像问题都不大，这也侧面证明了non-local操作这一本质才是提高网络性能的关键。  

为了简化操作，作者将g函数看成一个线性的embedding，只需要学习一个权重向量W<sub>g</sub>，这在实践中可以通过空间的1×1卷积或者时空的1×1×1卷积来实现。  

f就比较复杂和多样了。原始的non-local论文中建议使用Gaussian函数，这里的公式如下:  
![74-2](/images/daily paper/74-2.png)  

这里的指数是一个点乘的相似度，欧氏距离其实也可以，但是点乘在深度学习平台上实现起来更容易一些。采用该函数的时候C(x)是j个f函数值的和。  

也可以采取高斯函数的一个变体：Embedded Gaussian，公式如下：  
![74-3](/images/daily paper/74-3.png)  

这里的θ和φ都是两个embedding函数，学习两个权重函数W即可，这里C和上面一样，都是j个f函数值的和。  

这里比较有意思的是，作者注意到最近很火的自注意力模块其实就是采用该种形式的non-local operation的一种特例。具体来说，在这个函数中，对于一个给定的i，1/(C(x)) * f(xi,xj)变成了沿着j维度的softmax操作，整个操作可以写成y = softmax(x<sup>T</sup>W<sub>θ</sub><sup>T</sup>W<sub>φ</sub>x)g(x)，这也正是自注意力机制的公式。这就非常有意思了，作者认为他们的操作可以搭建一个自注意力机制到计算机视觉领域的桥梁，而昨天看的那篇SAN网络也正是借鉴了这里的思想，把自注意力机制应用到了CV领域的多个任务上。  

而作者经过试验后，发现这里基于softmax的attention操作好像并不是必要的，所以作者又提出了两个简化版本。第一个是点乘相似度函数，这里作者采用的是embedded版本，公式如下:  

![74-4](/images/daily paper/74-4.png)  

将C(x)设为N，N是x中位置的数量而不是f的和，这样有利于简化梯度运算。这里和embedded gaussian方式的唯一区别就是把softmax去掉了，这里softmax起到的是激活函数的作用。  

另一种方法是concatenation，这个更为简单粗暴，作者借鉴了视觉理解中的Relation Networks(不是few-shot的那个)，公式如下：  
![74-5](/images/daily paper/74-5.png)  

这里w<sub>f</sub>是一个权重向量，用于将concat起来的向量投影为一个标量。这里C(x)也等于N。  

在搞清楚f和g究竟是什么之后，就可以构建non-local模块了。作者定义一个non-local模块为 z<sub>i</sub> = W<sub>z</sub>y<sub>i</sub> + x<sub>i</sub>，这采用了残差的思想，从而保证该模块可以嵌入到任意预训练的任意位置而不需要打破原有架构。  

![74-6](/images/daily paper/74-6.png)  

上图就是一个embedded Gaussian版本的non-local模块，上面公式的pairwise操作可以被图中所示的矩阵乘法所代替。作者还提到了一个下采样的trick，即在φ,g后加上一个最大池化层，可以降低四分之一的pairwise操作，作者的所有实验部分都采用了这个trick。  

## Experiment  

作者的实验主要是在动作识别领域开展的。作者采用了C2D和I3D作为baseline，将non-local模块嵌入到了C2D和I3D中进行对比，在Kinetics上的结果如下图：  
![74-7](/images/daily paper/74-7.png)  

在Charades上的结果如下图:  
![74-8](/images/daily paper/74-8.png)  

作者还进行了二维的性能比较。这里作者没有在图像分类任务上进行实验，估计是性能提升不大，而是在目标检测和实例分割任务上进行了实验，在COCO数据集上的结果如下:  
![74-9](/images/daily paper/74-9.png)  

## Conclusion  

作者搞了一个新的操作non-local operation，将该模块嵌入到多个网络中，在多个任务上都能得到有效的性能提升。non-local的思想就是把全局的特征都考虑到，我觉得这个思想和DenseNet、DPN有点相似，不过随之带来的是巨额的计算复杂度。我盲猜这个模块的计算量应该不是一般的大，即使添加了作者的pooling trick，3/4的计算量还是足够服务器喝一壶的。这种偏暴力的方法当然有用，但是我感觉会将一大批没用的信息都计算在内。我们看过LSTM都知道，两端的时间步对中间的部分影响实在是很小，卷积核之所以有用和高效，也是把有限的计算资源放在了information-rich的邻近部分，不管是2D还是3D，所有的信息是否真的都有用，我觉得还是要画一个问号。长距离的依赖性当然要捕获，不过在时序上捕获的价值更大一些，这也能从作者的实验偏向性里看出，non-local模块在视频任务的表现更好一些。我觉得attention机制貌似比non-local的非attention部分要好一些，不管是设计上还是理论上都更为巧妙一点。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

---
layout: post
title: "Daily Paper 87: Linformer"
description: "Notes"
categories: [NLP]
tags: [Paper]
redirect_from:
  - /2020/07/01/
---

# Daily Paper 87: Linformer: Self-Attention with Linear Complexity  

## Introduction  

这篇文章是Facebook在六月中旬挂在arxiv上的，是对transformer在计算量方面的一种改进。作者认为虽然transformer的表现非常好，但是训练和部署这些模型需要花费相当多的时间，因为标准自注意力机制需要花费O(n²)的时间和空间复杂度。这里作者使用一个低秩的矩阵来近似标准自注意力机制，从而在时间和空间维度上把transformer的时间和空间复杂度变为O(n)。作者将改良后的transformer称为Linformer，在时间和空间复杂度大大降低的前提下取得了和标准transformer模型相当的性能。  

作者的灵感来源于发现自注意力是低秩的，更为确切的说，作者同时在理论和实验上证明了自注意力机制形成的随机矩阵可以被近似为一个低秩矩阵。因此作者就将原始的scaled dot-product attention分解为多个小的线性投影attention，这些操作的结合组成了一个对原始注意力的低秩因式分解。作者的复杂度对比如下:  
![87-1](/images/daily paper/87-1.png)  

## Backgrounds and Related Works  

首先回顾一下transformer的操作，下面是多头注意力中某一头的公式:  
![87-2](/images/daily paper/87-2.png)  

其中P可以看作一个上下文映射矩阵，但是计算P计算量是比较大的，由于需要完成两个n×d矩阵的相乘，所以时间和空间复杂度都是O(n²)，因此对于序列长度的二次时间/空间复杂度也成为了Transformer的瓶颈。  

最近也有一些方法来解决这一问题，比如mixed precision，知识蒸馏，sparse attention和Locally-sensitive hashing attention。也有通过提升优化效率的方式来减轻计算负担的方式。作者认为这些方式大多采用时间换空间的方式，作者力图找到一种时间复杂度和空间复杂度同时降低的方法。  

## Self-Attention is Low Rank  

这一部分主要证明了自注意力机制的上下文映射矩阵P是低秩的。作者首先使用了两个预训练的transformer模型RoBERTa-base和RoBERTa-large用于两个任务上：masked-language-modeling task on Wiki103和IMDB上的分类任务。作者将模型不同head的不同层的P分别进行了奇异值分解，将在10k个句子上的归一化的累积奇异值做了频谱分析，如下图左子图所示:  
![87-3](/images/daily paper/87-3.png)  

可以发现在每层、每个头、每个任务都有一个明显的长尾频谱分布，这说明矩阵P的大多数信息都可以从一开始的少数大奇异值中恢复。如上图的右子图所示，作者又画了一个第128大（一共512个）的奇异值位置上的归一化累积奇异值的热力图，结果显示高层的频谱分布相对低层来说更为歪斜，这说明在高层中，更多的信息体现在大奇异值上，P的秩更低。  

那么从实验角度证明过后，作者又通过理论角度证明了这一点，定理如下:  
![87-4](/images/daily paper/87-4.png)  

证明如下:  
![87-5](/images/daily paper/87-5.png)  

说实话，我连定理都看不懂，更别提证明了。这些复杂的符号和定理已经触及了没学过高代的我的知识盲区。这里先把证明放上，等我学完高代再来重新看看。  

## Model  

下图是作者的新attention:  
![87-6](/images/daily paper/87-6.png)  

可以看到作者的主要改动是在计算k和v的时候添加了两个线性投影矩阵E<sub>i</sub>和F<sub>i</sub>。具体来讲，作者首先把原始的n×d维度的k和v投影到k×d维度的投影k和投影v上，然后使用缩放点积注意力机制来计算一个n×k维度的上下文映射矩阵P，公式如下:  
![87-7](/images/daily paper/87-7.png)  

最后再计算每个头的context embedding，公式如下：  
![87-8](/images/daily paper/87-8.png)  

以上的操作都只需要O(nk)的时间和空间复杂度，因此如果能选择一个很小的投影维度k，使得k远小于n，那么就可以显著的减少时间和空间的复杂度花费。接下来的定理证明了，当k=O(d/ε²)（独立于n）的时候，可以使用线性自注意力，在ε误差的条件下逼近P·VW<sub>i</sub><sup>V</sup>。  

![87-9](/images/daily paper/87-9.png)  

在上面示意图的右侧，作者作了Linformer和标准Transformer对于不同序列长度的inference速度图，这里保持总体的tokens数量不变。结果显示标准Transformer在序列长度变长的时候速度明显变慢，而Linformer速度保持平缓，并显著小于Transformer。  

此外还有一些额外的trick可以在Linformer上使用，从而进一步优化表现和效率。首先是投影矩阵之间的参数共享，在不同的层和不同的头之间，E<sub>i</sub>和F<sub>i</sub>参数可以共享，共享还有三个级别：第一个是headwise sharing，也就是说在每层的不同head间共享；第二个是key-value sharing，也就是在headwise sharing的前提下，让E<sub>i</sub>和F<sub>i</sub>也相等；第三个是layerwise sharing，这个就好理解了，所有的层的所有的head都一样，E<sub>i</sub>和F<sub>i</sub>也一样，一共就一个矩阵。举个栗子，对于一个12层12头的模型，这三种方式的矩阵数量分别是24,12和1。  

其次是Nonuniform projected dimension.对于不同的层和头，可以选择不同的投影维度k。由于之前提到，在更高层的头可能会有更歪斜的频谱分布，也就是更低的秩，因此在更高的层中选择更小的投影维度k会更好一些。  

最后是general projections，也就是可以选择不同的低维投影方法来代替单一的线性投影，比如可以使用最小最大池化，或者卷积核维度和步长均为n/k的卷积，不过卷积会带来额外的训练参数。  

## Experiment  

首先是对于n和k的研究，n代表序列长度，k代表投影维度，结果如下:  
![87-10](/images/daily paper/87-10.png)  

其次是和其他模型的对比，结果如下:  
![87-11](/images/daily paper/87-11.png)  

最后是性能的提升，结果如下:  
![87-12](/images/daily paper/87-12.png)  

## Conclusion  

总体来讲，作者从理论和实践两方面证明了线性注意力可以代替自注意力模块，并用实验验证了替代模型能够在获得和Transformer相当表现的同时显著的减少时间和空间的复杂度。我个人认为这篇文章可以和之前看过的一些用self-attention替代卷积的文章结合起来，尝试一下能不能用这种更为轻量化的方式来代替卷积操作，实现一个全注意力模型的轻量化网络。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

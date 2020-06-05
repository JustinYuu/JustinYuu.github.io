---
layout: post
title: "Daily Paper 77: Attention-CNN"
description: "Notes"
categories: [CV-Attention]
tags: [Paper]
redirect_from:
  - /2020/06/04/
---

# Daily Paper 77: On the relationship betweeen self-attention and convolutional layers  

## Introduction  
  
今天的这篇是带制作，是EPFL的Jaggi组发在ICLR2020上的。这篇文章最大的贡献是探究了自注意力机制和卷积层之间的关系，并尝试使用自注意力机制来代替卷积层。19年Ramachandran的Stand-alone self-attention in vision models一文已经证明了自注意力能够完全替代卷积层，并能在视觉任务上实现SOTA的表现。此外之前看的CVPR2020的SANet，也使用了完全基于自注意力的网络来实现了多个CV任务上的SOTA，那么这就引发一个问题，注意力层和卷积层的操作是否是一样的呢？作者的这篇paper就证明了，注意力层能够实现卷积操作，并且事实上经常在实践中学习这么做。特别的，作者还证明了一个有足够多的头的多头注意力机制能够达到任何卷积层的表现效果。作者的数值实验也证明了自注意力层对于像素网格模式的处理也和CNN层类似，这也证实了作者的观点。  

一切的一切还要追溯到Transformer上，在基于transformer/multihead attention的众多模型，比如BERT，GPT-2等等血洗了NLP和语音识别的榜单之后，众多学者又将眼光投向了CV领域。而之所以Transformer能移植到CNN上去，还要多亏了注意力机制。注意力机制的目的是为了获得长距离的依赖性，而这里采用的是注意力机制中的自注意力机制，也就是使用注意力分数来衡量一个序列中任意两个词语的关系，从而基于这些词语中的注意力权重值的高低来进行每个单词表示的更新。那么注意力机制最初应用到计算机视觉领域，还要追溯到SE-Net的channel-based attention和何凯明的Non-local Network，这些之前也都介绍过了，通过attention，来探究channel维度上和空间维度上的权重关系，也算是强行把序列间的关系转移到了CNN架构内部中。那么到这里，attention的本意还是为了提高CNN网络的表现，换言之attention只是起到了CNN架构的辅助作用，那么谷歌大脑的Ramachandran在2019年发表了一篇非常重要的文章，创造性的将自注意力代替了CNN，成为了网络的骨架部分，这篇文章接下来也会详细的介绍一下。前几天读过的SANet也是类似的文章，使用自注意力模块来实现了CNN的特征整合和特征转换两部分。  

那么既然有了用自注意力机制来代替CNN的成功案例，作者就在想，自注意力机制是真的按照CNN的做法去做的呢，还是使用另外一种新的方法来实现，只是恰好有了很好的表现而已的呢？这里要提一下SANet和这篇paper几乎是同时期的，也就是说作者探究的是基于Ramachandran这篇文章。SANet已经读过了，是完全模仿CNN的作用，而这里作者要探究的是Ramachandran的自注意力机制究竟是模仿CNN还是另有方式。从理论的角度上来看，transformer其实有能力去模仿任何函数，包括CNN，毕竟CNN的卷积本身也是一个函数。事实上，Perez等人在2019年已经证明了带有positional encoding的注意力架构在一些强理论假设下是图灵完备的，那么既然都图灵完备了，一个小小的卷积操作当然不在话下。但是该研究并没有明确的揭露到底是如何实现的，只是证明了能够实现，那么作者的motivation也就出来了，即找到自注意力机制到底是如何实现卷积操作的。  

作者的contribution有二，从理论和实验两方面证明了自注意力机制可以并且确实做了和卷积层操作类似的事。从理论层面，作者提出了一个有建设性的证据，证明了自注意力层可以表达任何卷积层能表达的特征，并证明了单个多头注意力机制加上positional encoding可以通过重参数化以表达任意卷积层所表达的特征。从实验层面，作者证明了Ramachandran的文章的确在每一个query像素上学习了网格化的特征，这也和卷积操作的实质类似。  

## Background on attention mechanisms for vision  

文章先介绍了一下多头注意力机制，这里就不赘述了。将注意力从文字序列移植到图像的像素中，需要做到1维序列到2维序列的转变，那么q和k就是二维的像素图，其维度为W×H。那么为了保证公式和1维的时候相同，作者使用向量来代替标量进行标注，比如对于像素点p=(i,j)，就用X<sub>p,</sub>来代替X<sub>i,j,</sub>，那么像素级别的多头注意力值可以用下列公式表示:  
![77-1](/images/daily paper/77-1.png)  

此外，上面提到，这里还需要进行transformer中的positional encoding，这里一般有两种方法：绝对编码和相对编码。对于绝对编码而言，每一个像素p都有一个固定的或者可学习的向量P<sub>p</sub>，那么自注意力分数的计算就可以被分解为下述公式:  
![77-2](/images/daily paper/77-2.png)  

至于相对编码，只需要考虑query和key像素之间的位置不同，而不需要考虑key像素的绝对位置，公式如下:  
![77-3](/images/daily paper/77-3.png)  

其中δ指的是key和query的位置之差，u和v是可学习矩阵，对于每一个head参数是不同的，r<sub>δ</sub>代表每一个偏移量δ的相对位置编码，对于每一个层和每一个头都是共享的。此外，权重向量分为了两部分，分别是不戴帽子的W<sub>key</sub>，属于原始输入，和戴帽子的W<sub>key</sub>，属于像素的相对位置。  

## Self-Attention as a Convolutional Layer  

接下来开始讲重点了。首先是定理1：一个有N<sub>h</sub>个头，每个头维度是D<sub>h</sub>维，输出维度是D<sub>out</sub>的多头注意力机制和一个维度D<sub>p</sub>≥3的相对位置编码能够表达任意卷积核为√N<sub>h</sub>×√N<sub>h</sub>，输出维度为min(D<sub>h</sub>, D<sub>out</sub>)的卷积层。  

该定理是通过手动选择多头自注意力层的参数使其能够达到卷积层的表现来实现的，具体来讲，每一个自注意力头的注意力分数应当具有一个不同的相对偏移量，而偏移量应当在下述集合内，K代表卷积核的尺寸:  
![77-4](/images/daily paper/77-4.png)  

具体的条件在下面的引理1中阐述，而引理2证明了前面提到的条件满足相对位置编码，作者称之为二次编码，公式如下:  
![77-5](/images/daily paper/77-5.png)  

其中可学习的参数△<sup>(h)</sup>和α<sup>(h)</sup>决定了每一个头内的注意力的中心和宽，此外δ是固定的，表示query和key像素之间的相对偏移。  

作者强调以上的编码并不是唯一符合引理1的编码方式，事实上从神经网络中学习到的相对编码也能够满足引理1的条件。然而，上述定义的编码在尺寸方面非常有效，因为仅Dp = 3维就足以对像素的相对位置进行编码，同时还达到了类似或更好的经验性能（比学习的更好）。  

上述的定理证明了自注意力机制能够实现卷积操作，但是我们大家都知道，在实际应用中，卷积网络并不只有卷积核这一个参数，还包括其他的超参数，那么自注意力机制要想完全代替卷积操作，还需要实现其他的卷积超参数，比如padding，stride和dilation。对于padding，多头注意力层使用默认的“SAME” padding方式，使输出的维度和输入的维度相同，这个应该比较好实现，只需要将输入的图片的每一边都进行K/2(下界)个0的padding即可。对于stride，这其实和pooling的效果类似，定理1的stride为1，只需要在自注意力层后面跟随固定步长的池化层即可模拟任意大小的步长。最后是扩张卷积，每个头都可以在任何像素偏移处插入一个值，从而形成膨胀的卷积图。  

最后是1维卷积的情况，定理1可以直接延伸到1维卷积的情况下，卷积层K=N<sub>h</sub>，输出维度是min(D<sub>h</sub>,D<sub>out</sub>),使用D<sub>p</sub>≥2的位置编码。不过作者并没有做实验来证明1维的自注意力机制表现是不是和1维卷积表现相同，只是从理论层面上证明了可行性。  

## Proof of Main Theorem  

最后是整篇文章的重点，即对定理1的证明。定理1的证明分为两个引理，也就是上面提到的引理1和引理2.  

### Lemma 1  

引理1：一个多头自注意力层有N<sub>h</sub> = K²个头，D<sub>h</sub>≥D<sub>out</sub>，定义f:\[N<sub>h</sub>] → △<sub>K</sub>为一个头到偏移量的双射映射。那么，假设对于每一个head，都有下述公式:  
![77-6](/images/daily paper/77-6.png)  

那么对于任意卷积核尺寸为K×K，输出通道维度为D<sub>out</sub>的卷积层，都存在{W<sub>val</sub><sup>(h)</sup>}<sub>h∈\[N<sub>h</sub>]</sub>，使得对于每一个X都有MHSA(X) = Conv(X)。  

引理证明：我们知道多头自注意力的公式如下:  
![77-7](/images/daily paper/77-7.png)  

每一个头的V矩阵W<sub>val</sub><sup>(h)</sup>和每一个块的投影矩阵W<sub>out</sub>(D<sub>h</sub>×D<sub>out</sub>)是学习得到的，假设D<sub>h</sub>≥D<sub>out</sub>，那么我们可以把每一个头的两个矩阵换成可学习的矩阵W<sup>(h)</sup>，那么对于某一个特定的多头自注意力的输入像素q来说，公式就变成了下面的样子：  
![77-8](/images/daily paper/77-8.png)  

由于引理的限制，对于第h个注意力头，当k=q-f(h)的时候注意力概率是1，否则是0，那么对于像素q的注意力层输出来说，公式就又成了下面的样子:  
![77-9](/images/daily paper/77-9.png)  

当K=√N<sub>h</sub>时，上述公式可以等同于卷积操作。  

一般来说，在基于transformer的应用中，D<sub>h</sub> = D<sub>out</sub>/N<sub>h</sub>，因此D<sub>h</sub><D<sub>out</sub>，那么W<sup>(h)</sup>可以看成是秩为D<sub>out</sub>-D<sub>h</sub>的矩阵，这不足以表达具有D<sub>out</sub>个channel的卷积层。不过，虽然秩小于D<sub>out</sub>，但是MHSA(X)的输出仍然是D<sub>out</sub>维度的，从这里边选出任意D<sub>h</sub>个输出，能够表达出任意具有D<sub>h</sub>个channel的卷积层输出，所以作者给定理1加了一个约束条件，即卷积层的输出channel应该是min(D<sub>h</sub>, D<sub>out</sub>)。在实际操作中，作者建议把多个D<sub>h</sub>的头concat成为D<sub>out</sub>，而不是把D<sub>out</sub>维度的输出split为多个head，这样可以精确的重参数化，避免产生无用的channel。  

### Lemma 2  

引理1证明了卷积操作是可以用多头自注意力等价的，引理2证明了存在一个相对编码方案{r<sub>δ</sub>∈R<sup>D<sub>p</sub></sup>}<sub>δ∈Z²</sub>，其中D<sub>p</sub>≥3，该方案包含参数矩阵W<sub>qry</sub>, W<sub>key</sub>, W^<sub>key</sub>，u，同时D<sub>p</sub>≤D<sub>k</sub>，使得对于每一个△∈△<sub>k</sub>，都存在一些在△上约束的向量v，使得如果k-q=△，softmax(A<sub>q,:</sub>)<sub>k</sub>=1，否则等于0.  

引理2证明：  

作者通过举例来证明D<sub>p</sub>=3的时候，相对编码方案满足要求的注意力概率。由于注意力概率对于输入的张量X是独立的，所以设置W<sub>key</sub>和W<sub>qry</sub>的初始值为0，那么上面相对位置编码公式就只剩了最后一项了，那么把W^<sub>key</sub>(D<sub>k</sub>×D<sub>p</sub>)设为单位矩阵（进行行填充），使得A<sub>q,k</sub> = v<sup>T</sup>r<sub>δ</sub>, δ:=k-q。此外上面提到作者假设D<sub>p</sub>≤D<sub>k</sub>，所以没有来自r<sub>δ</sub>的信息丢失掉。然后我们假设有下列公式成立:  

![77-10](/images/daily paper/77-10.png)  

c是常数，在上面的公式中，A<sub>q,:</sub>最大的注意力分数是-αc，此事位置达到了A<sub>q,k</sub>，此时δ=△。另一方面，系数α可以用来任意比例的缩放A<sub>q,△</sub>和其他注意力分数之间的区别。  

那么对于δ=△，我们有以下公式:  
![77-11](/images/daily paper/77-11.png)  

对于δ≠△，lim<sub>α→∞</sub>softmax(A<sub>q,:</sub>)<sub>k</sub>=0，因此满足引理2的要求。  

此外还需要证明的是存在v和{r<sub>δ</sub>}<sub>δ∈Z²</sub>使得上述假设公式真的成立。把假设公式的右边展开，得到  
![77-12](/images/daily paper/77-12.png)  

那么如果我们把v和r设成  
![77-13](/images/daily paper/77-13.png)  

我们就会得到  
![77-14](/images/daily paper/77-14.png)  

那么当c=-\|\|△\|\|²时，条件满足，因此论证完毕。  

此外，对于任意像素点的准确表示要求α，或者Wq和Wk能够达到任意大，同时当α增加的时候，其他像素的注意力权重指数级收敛为0。但是在实际操作中，一般依靠有限精度算法，找到一个能够满足实验所需的常数α即可，比如最小的正float32标量是10e-45，那么把α设成46就足够获得hard attention了。  


---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

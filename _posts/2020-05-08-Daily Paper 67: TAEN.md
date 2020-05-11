---
layout: post
title: "Daily Paper 67: TAEN"
description: "Notes"
categories: [CV-Meta]
tags: [Paper]
redirect_from:
  - /2020/05/08/
---

# Daily Paper 67: TAEN: Temporal Aware Embedding Network for Few-Shot Action Recognition  

## Introduction  

这篇是IBM和Technion在4月底挂在arxiv上的，和之前的TARN只有一字之差，做的也是小样本动作识别的工作。TAEN的全称是Temporal Aware Embedding Network for Few-Shot Action Recognition, 这里比较好玩的是他在两个任务上进行了数据集验证，分别是视频分类和时序动作检测上，这正好是我比较感兴趣的两个任务，该算法同时在这两个任务上实现了SOTA。  

作者的idea来源于对于动作识别方式的思考。最近很火的双流网络，比如C3D,I3D，通常都是处理短的视频片段，比如16个连续帧，这样就会损失掉一些含有复杂动作的长期轨迹，因此大家一般使用子动作分解的方法作为补救措施。这些子动作是视频内一个长期动作的短时序片段，而作者将这些子动作看成一系列连续的时序片段，顺序由它的时序顺序来决定。而作者的方法试图使用这些子动作和它们的时序顺序来将动作表征为度量空间内的一个轨迹，最终能够在一个与一系列动作相关的深度特征空间内学习运动的类型，如下图所示:  
![67-1](/images/daily paper/67-1.png)  

这个图画的更像是整体算法训练的流程图，我认为这里最大的创新点在于他们使用了动作的子动作生成embedding，然后根据特征空间内的子动作轨迹的相似度来进行分类。这个表示能够把语义、细化的时序表示和时序在一个粗糙的细粒度程度上同时表达出来。和其他方式不同的地方在于，其他论文学习到的embedding并没有明确的指出是什么类型的embedding，或者说对于embedding和特征空间内到底存了什么东西并没有很明确的阐述，而这篇文章明确的指出学习到的embedding是子动作的时序序列，在特征空间内比较的是轨迹的相似度。  

作者认为他们的contribution有三点：首先提出了一个新的度量学习方式，其次针对这个衡量轨迹相似度的学习方式提出了一个新的loss，最后再小样本动作识别的视频分类和时序动作检测上取得了STOA的效果。我觉得真正的contribution是把动作识别和时序动作检测的子动作检测和iDTF转移到了小样本中，这应该是属于动作识别+时序动作检测与小样本的结合体。  

## Related Works  

这篇的related works还是很有意思的，对于子动作检测部分，作者提到了比较著名的ActionVLAD，将子动作和深度神经网络合在一起应用到动作识别任务中。此外还提到了比较关键的iDTF，即hand crafted dense trajectory feature和基于iDTF的时序动作检测方法Real-time temporal action localization in untrimmed videos by sub-action discovery(BMVC2017)。  

对于小样本动作识别部分我就不多说了，我博客里介绍的那几篇已经是近几年所有的研究了。作者只引用了已发表的两篇文章CMN和TARN，当然这个领域还有没发表的两篇，分别是Daily paper 56中1月份的一篇和Daily Paper54中去年六月份的一篇，不过这两篇和作者的算法也没什么关联。  

对于小样本时序动作检测我认为这是一个冷门到不能再冷门的领域。据我所知现阶段有两篇类似的研究，一篇就是作者所唯一引用的CVPR2018的One-shot action localiztaion by learning sequence matching network，这篇文章使用matching network的思想来处理这一问题，另一篇更早，是ACM MM2017的SSAD网络。我觉得这个领域真的是大有前景，因为前面两篇发表在2017和2018年，而这几年时序动作检测已经出了无数种新方法，目标检测领域也多了无数种新的算法可供移植。就算完全使用相同的小样本方式，就更新时序动作检测方式本身我估计也能提高性能挺多。  

## Method  

整体的流程图如下图所示:  
![67-2](/images/daily paper/67-2.png)  

如图所示，整体的流程分为两步：首先学习一个特征空间，在这个特征空间内动作能够以可区分的轨迹来表示，轨迹动作按照时间顺序排列，每一个子动作对应一个轨迹元素。然后再根据新视频的轨迹特征来进行分类。  

### Embedding architecture  

首先是生成embedding的骨架。作者沿用了RepMet架构，从而能够并行的训练embedding和子动作representatives(这里作者用centroids代指)。对于RepMet我之前没看过，这是属于小样本目标检测领域的文章，我找了一篇[知乎文章](https://zhuanlan.zhihu.com/p/85704258)，可以参考一下。通过该架构，可以并行的学习到每一个class的多种表示，在这里也就是子动作，然后通过对embedding网络的训练，从而生成最优的embeddings和centroids。这里解释一下centroids，作者把深度度量学习(DML)空间中的轨迹trajectory定义为一系列子动作表示的时序集合，对于子动作表示，作者将其代称为centroids。在我的理解下，时序的centroids就是trajectory。作者设计了一个新的loss函数用于辨别embedding空间中的trajectories，这个后面再说。  

网络架构的输入是一系列视频特征，这是通过预训练的C3D或者I3D处理的，特征X的维度是T×d<sub>feat</sub>，其中前者是视频片段的数量，也就是子动作的数量，后者是特征的维度。作者将视频分为a个片段，然后对于每一个片段都使用平均池化来计算他们的特征表示X<sub>p</sub>，维度为a×d<sub>feat</sub>。到这里我有点晕，我不太明白X和X<sub>p</sub>之间的关系是啥，好像是一个东西？然后作者使用X<sub>p</sub>作为DML网络的输入，DML网络由一系列FC和ReLU组成，最后的输出embedding E的维度是a×e，e是embedding的维度，一般来说e≤d<sub>feat</sub>。  

### Sub-action centriods  

为了能够对多子动作进行联合训练，作者还建立了一个secondary branch，如上面图示的上半部分所示。这一个分支能够计算每一个动作类的centroids，centroids通过一个常数标量导入到FC层中产生，FC层的参数维度为C×a×e，C是训练集中动作的类别数量，a是一个动作中的子动作数量，e是embedding的维度。多个centroids组成的这个C×a×e的class representatives，就类似protonet中的prototype。  

### Loss function  

至此我们得到了原始数据生成的embedding E，以及sub-action centroids R<sub>c</sub>，接下来要做的是使用这两者来计算embedding空间内轨迹间的距离，公式如下：  
![67-3](/images/daily paper/67-3.png)  

这个公式其实就是把为每一个类的R都和E逐子动作计算一下距离，然后再相加取平均，这里距离函数使用的是超球下的余弦相似度。  

loss函数由三部分组成，第一部分叫做affiliation loss，该loss可以最小化一个视频的embedding E中子动作距离他们的centroids之间的距离，我认为就和protonet的思想类似，公式如下:  
![67-4](/images/daily paper/67-4.png)  

第二部分叫做motion loss，用来衡量轨迹梯度的偏差，通过连续的子动作间向量的改变来估计，具体方式是把两个连续子动作间的R和E相减然后做内积，然后把a-1个结果求和，公式如下:  
![67-5](/images/daily paper/67-5.png)  

作者认为该loss中子动作间的向量归一化操作能够给embedding空间带来更多的子动作多样性，同时能够进一步有利于模型和测试的动作向量的对齐。  

第三部分叫做diversity loss。该loss的目的是为了防止每一个动作类别中不同的子动作表示在embedding空间内的一点塌陷，我估计这应该是训练的时候出了问题加上的。由相近的点决定的短轨迹能够带来更没有分辨力的特性，那么为了避免这一情况，作者引入了diversity loss，使得相关的子动作在特征空间的位置稍远一点，从而避免不同的子动作有相同或者相当接近位置的情况。这可以通过惩罚高相关性的centroids就可以实现，公式如下:  
![67-6](/images/daily paper/67-6.png)  

最后总体的Loss就由这三个loss加权平均得到，三个权重是超参数。在测试的过程中，只需要对动作间的相似度进行测量，测量的结果是一个离散的point-wise similarity和motion间的加权距离，不需要加上diversity loss，公式如下:  
![67-7](/images/daily paper/67-7.png)  

## Experiment  

作者的分类特征提取器是在Sports-1M上预训练的C3D，时序动作检测是在ActivityNet 1.2上的非测试集上预训练的I3D网络。测试采用episode training的方式。对于视频分类任务，每一个标注的视频都被映射到一个embedding空间中的轨迹中，取最邻近的trajectory对应的类别。对于时序动作检测来说，作者建立了一个时序RPN，整体的流程如下图所示:  
![67-8](/images/daily paper/67-8.png)  

图分为三层，最上层是使用训练过的DML和相同的flow来获得所有support set videos的embeddings和相关的子动作，第二层是分类层，这个很简单，只需要将其结构为a个子动作，然后映射到DML空间中，创建一个轨迹，进行轨迹最邻近就可以了。第三层是定位层，时序动作检测的方法由于之前没接触过，这里多提两句。特征空间里面是trajectory，每一个support class的trajectory都使用和分类任务相同的方式来计算，但是由于query是未标注的任务，所以需要把query video先分解为时序区间，这需要用到目前标准时序动作提案方法，之后再将提案片段依据轨迹距离进行分类和打分。一个动作提案属于某一个特定动作类的概率如下列公式所示:  
![67-11](/images/daily paper/67-11.png)  

其中σ是一个控制概率测量标准差的超参数，然后使用该概率计算分数，公式如下图:  
![67-12](/images/daily paper/67-12.png)  

分类任务的结果如下图，确实超越了之前的两个baseline:  
![67-9](/images/daily paper/67-9.png)  

时序动作检测的结果如下图，可见也达到了SOTA。  
![67-10](/images/daily paper/67-10.png)  

消融实验的结果略。  

## Conclusion  

这篇文章确实不错，首先把当今小样本动作识别的发表过的文章的性能打破了，然后还用相同的方法实现了小样本的时序动作检测，相当于一个算法做了两个任务。在算法层面，没有将目光聚焦在现在大家都喜欢使用的注意力机制上，也没有聚焦在时序信息的处理上，而是创造性的把特征空间的内容变成了运动的轨迹，并不是对RGB图像序列的相似度进行比较，而是直接对轨迹进行比较，而轨迹正是对一个动作来说最为关键的、最有辨别力的部分。最终的效果也表明了这种方式是有效的。作者在训练的时候并没有采取episode training的方式，采用三个loss进行联合训练，在算法的复杂性上还不算太过复杂，总之是一篇好文章。  

此外这篇文章还给小样本视频领域的研究提出了新的思路，对于轨迹本身的的研究，能不能借用当今小样本的一系列方法进行研究呢？比如把注意力机制应用在轨迹相似度的比较上，把跨模态的信息应用在轨迹比较中等等。同时这篇文章还给许久都没人做的小样本时序动作检测方向提供了一个新的baseline，也算是为这个冷门领域做出了贡献。看完这篇文章我就一直在想，这么好的点子我怎么就想不到呢？搞了半天embedding，怎么就不想想embedding的究竟是什么玩意，能不能用更好的来替代呢？还是自己水平太次，不过这也说明了看起来研究方向很局限的任务还是有很多可供挖掘的新方向，也不一定需要聚焦在这一两个大家都在做的方向上进行这一任务的研究。其实对于轨迹和子动作的研究，也不是作者一拍脑门想起来的，而是在动作识别领域早就有过应用的，作者强就强在，他知道这个东西，并能稍作修改搬到小样本的任务中来，进一步经过复杂的训练过程得到了不错的结果。  


---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

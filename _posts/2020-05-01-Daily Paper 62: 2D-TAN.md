---
layout: post
title: "Daily Paper 62: 2D-TAN"
description: "Notes"
categories: [MMML-Event_Localization]
tags: [Paper]
redirect_from:
  - /2020/05/01/
---

# Daily Paper 62: Learning 2D Temporal Adjacent Networks for Moment Localization with Natural Language  

## Introduction  

不知不觉已经到了五月了，然而我还是和二月份的自己一样废物，并没有做出什么像样的东西来。三月份看了小样本动作识别的很多论文，四月份写了一个月的代码，确定了接下来要做的任务audio-visual event localization，也看了这个领域的所有文章。五月份总归要在这个领域做点东西了。  

这篇paper是Rochester和微软发表在AAAI2020上的。我越来越感觉AAAI真是一个神奇的会议，这个会议既能出神文，又能出水文，真是个兼容并包群魔乱舞的会议。这篇paper我感觉还是不错的，作者开创性的使用2D的Temporal Adjacent  Network来进行NLP的Moment Localization，并在Moment Localization任务上取得了不错的效果。  

作者研究的任务叫做Moment Localization with natural languange，指的是根据一句给定的自然语言指令来从未修剪视频中检索一个时序的片段。现有的方式大多都是采用滑动窗口的方式选取moment candidate，然后依次和query sentence进行比对，作者认为这样只是单独的考虑各个moment candidates，而忽略了他们的时序依赖性，所以要想把事件发生的具体时间boundary准确定位是很难的。所以作者提出了新的2D-TAN网络，其核心思想是将目标时刻定位在一个2维的时序图上，时序图上包含有不同的长度各异的视频时刻，表示他们的邻接关系。作者的2D-TAN可以处理更多的moment语义信息，从而判断该时刻是否与其他时序片段相关，与此同时，相邻的moment可能有重叠的内容，但是可能会表达不同的语义，2D-TAN将其考虑在内，从而能够学习到更有辨别力的特征来区分它们。  

## Method  

2D-TAN主要分为三步：language representation, video representation和moment localization。整体流程如下图:  
![62-1](/images/daily paper/61-1.png)  

### Language Representation  

首先看language representation，作者使用GloVe word2vec模型来处理句子生成词嵌入，然后使用一个三层的双向LSTM来处理词嵌入，使用最后一个hidden state来作为输入句子的特征表示。  

### Video Representation

接下来是video representation，作者将整个视频分成若干小的video clips，每一个video clip由T个帧组成。接下来对这些video clips进行fixed-interval采样，获得N个video clips。对于每个video clip，使用预训练的CNN模型进行特征提取。这里作者为了获得一个更紧凑的representation，把提取到的特征过了一遍FC层，最后的特征维度为d<sup>V</sup>，特征用f<sup>V</sup>表示。  

接下来使用f<sup>V</sup>来构建moment candidate的feature map。在之前的研究中，一般采用的方法是pooling或者stacked convolution。在这里作者沿用了pooling的方法，对于每一个moment candidate，都对一个特定的时间跨度内相关的clip features进行最大池化，获得最终的feature f<sup>M</sup><sub>a,b</sub>，其中a,b是video clips的起始点和结束点的index。不过作者也实验了stacked convolution的方法，并进行了比较，应该是pooling的效果要好一点。  

与之前的方法不同的是，这里并不是直接在单独的video moment上进行操作，而是重新将采样的moment重构到一个2D时序特征图上，记作F<sup>M</sup>，维度为N×N×d<sup>V</sup>。前面的两个维度N代表开始和结束的clip indexes，第三个维度d<sup>V</sup>代表特征维度。举个例子，如果一个moment从clip v<sub>a</sub>到clip v<sub>b</sub>，那么feature就会定位在F<sup>M</sup>\[a,b:]上。由于2D的特征图有两个轴，而显而易见a一定要小于等于b，所以时序轴有一半是无效的，即a＞b的那部分，只有在起始时间小于等于结束时间的那一部分才是有效的，如下图所示，蓝色部分和其右上方的部分是有效的，蓝色部分代表选取的moment candidates。  
![62-2](/images/daily paper/61-2.png)  

那么蓝色的部分是怎么选择的呢？一个简单的方法是把所有可能的连续video clip都作为candidates，但是这种简单粗暴的方式可能会增加计算负担，所以作者提出了一个稀疏采样策略，其核心思想是去除和已选的candidates有大量重叠的冗余moments。如上图所示，只有蓝色的点是moment candidates。具体来讲，作者对短时长的moment密集采样，而随着moment的时间时长变长，逐渐的增加采样的间隔。从上图可以看到，在对角线附近的moment起始点和终止点很接近，对于这部分短时长的video clip密集采样，而对于长时长的moment采样的频率就逐渐减小了。具体的采样规则是当N≤16，也就是总长度≤16的时候全部采样，当N>16的时候，只有满足下列条件的时候才可以采用：  
![62-3](/images/daily paper/61-3.png)  

如果G(a,b)=1，那么该moment就入选了，反之则不选。这种采样方式可以大大减少moment candidates的数量，并大大减少计算的复杂度。  

### Moment Localization  

最后是moment localization部分，也就是从上面确定的moment candidates中选择最合适的moment。这一步又分为三步：multi-modal fusion, context modeling和score prediction。  

首先是2D时序特征图F<sup>M</sup>和编码的语句特征f<sup>S</sup>的特征融合。作者使用一个FC层将两个跨模态的特征投影到一个联合子空间内，然后使用Hadamard乘积和l2均一化进行特征融合，公式如下：  
![62-4](/images/daily paper/61-4.png)  

w<sup>S</sup>和W<sup>M</sup>是FC层中需要学习的参数。接下来使用融合后的2D特征图F来建立Temporal Adjacent Network。网络架构非常简单，由L个卷积层组成，kernel size=K。网络的输出和输入的特征图经过zero padding后维度相同，这样可以使得模型能够逐渐的感知更多邻近moment candidates的语义，从而学到moment candidates之间的不同之处。此外，网络的感受野很大，这样可以观察到整个视频和句子的整体内容，从而学习到时序依赖性。值得一提的是，2D特征图中有zero padding，而在卷积计算的时候只计算有效的部分，不把zero-padding的部分参与到计算中。  

最后进行结果的预测，通过一个FC层和一个sigmoid函数生成一个2D的score map。最大的值就是对应最好的moment。  

### Loss Function  

这篇文章的loss函数很有意思。作者使用了IoU作为监督的信号，而不是单纯的二元判准。对于每一个moment candidate，作者都计算它和ground-truth的 IoU o<sub>i</sub>，然后进行下列操作：  
![62-5](/images/daily paper/61-5.png)  

这里两个阈值t<sub>min</sub>和t<sub>max</sub>是事先设定好的，使用二元交叉熵误差进行训练。  

## Experiment  

数据集采用Charades-STA，ActivityNet Captions和TACoS。直接放结果好了，在Charades-STA的结果如下图：
![62-6](/images/daily paper/61-6.png)  

在ActivityNet Captions的结果如下图:  
![62-7](/images/daily paper/61-7.png)  

在TACoS的结果如下图：  
![62-8](/images/daily paper/61-8.png)  

消融实验的结果就略过了。  

## Conclusion  

这篇作者最大的创新点就在于使用2D的时序图来处理Moment Localization with natural languange任务，并得到了不错的效果。我认为该方法也能够应用在时序动作检测和audio-visual event localization任务上。二维时序图是一个可研究的方向。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

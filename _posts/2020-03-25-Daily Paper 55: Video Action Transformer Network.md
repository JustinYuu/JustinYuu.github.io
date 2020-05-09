---
layout: post
title: "Daily Paper 55: Video Action Transformer Network"
description: "Notes"
categories: [CV-Meta]
tags: [Paper]
redirect_from:
  - /2020/03/25/
---

# Daily Paper 55: Video Action Transformer Network  

## Introduction  

这篇文章是CMU、DeepMind和VGG组合作发表在CVPR2019上的，主要contribution是把transformer用于非小样本动作识别，实现了动作识别的SOTA。这里的attention主要是用来结合特定的人的相关区域和所在的语境，从而识别动作。multi-head中的每一个head都用来计算一个clip embedding，用来着眼于当前动作下不同的部分，比如手、眼、其他人等。  

动作识别一直是一个很复杂的东西，除了要观察目标人物而外，还要观察其周围的环境，与之互动的人和物体等，这就需要通过注意力机制来进行两者之间的联系。作者自然而然的想到了Transformer，使用注意力机制来将上下文的联系考虑在内。作者的模型主要由以下几部分组成：一个时空I3D模型用来提取特征，一个区域提案网络RPN用来定位动作执行人，两者的结果作为Transformer的输入，整合其他人和环境的信息。transformer主要聚焦在人脸和手，由于这是进行动作的主要部位，一般会隐含更多的信息。这些东西都没有经过显式的监督，而是在动作分类的训练过程中获取。  

作者在Atomic Visual Actions数据集上进行了实验，该数据集需要检测许多人，通过多个bounding box才能判断具体的动作是什么。作者在这上面的表现也不错，这个留到实验部分再讲。  

## Model  

作者的模型需要探测任意时间点的所有人和他们所在做的动作。整个模型的流程是一个base-and-head网络，类似于Faseter R-CNN一样。base模块，也叫trunk模块，使用了一个3D卷积架构来生成当前时间步内人的特征和区域提案，head模块使用与每个区域相关的特征来预测动作，并回归一个更紧的bounding box。这里要注意的是RPN和边界框都是动作无关的。head模块使用trunk模块生成的特征图和RPN提案，使用RoIPool操作来生成一个和每个区域相关的特征表示。该特征紧接着用来将边界框分类，从C个动作类别和1个背景类中选出对应的类别，并回归一个4维的偏移向量，来将RPN提案转化为一个目标人更紧的边界框。  

### Base network architecture  

作者首先将原始视频 提取为T帧(一般为64)的片段，然后对于每一个关键帧都进行3秒的上下文编码。之后作者使用一系列卷积层构成的网络，也就是所谓的trunk进行编码。  

在具体实验中，作者使用了在Kinetics-400上预训练的I3D网络的初始层，从Mixed_4f层中提取特征，将T×H×W的输入下采样为T/4×H/16×W/16。作者将特征图的时序中间帧切出来，然后传到一个区域提案网络RPN中。RPN生成了多个可能的人的边界框，作者选取R(作者定义R=300)个最高分数的边界框进行进一步回归，回归到一个更紧的边界框中，使用head网络进行动作分类。  

### Action Transformer Head  

接下来是作者的Transformer架构。该部分使用了RPN的person box作为query，用来定位相关的region，然后整合了clip上的信息来进行动作分类。  

Transformer之前介绍过，主要是通过一个多头注意力机制来将当前的feature和序列中的其他feature相比较。当前的feature经过线性变换变为query(Q)，其他的feature变为key和value(K和V)，一般情况下Q和K都是更低维的。query和V进行注意力加权，得到query的输出，而注意力权重来源于Q和K的乘积。在NLP中，query一般是当前需要翻译的词语，而K和V是当前词语的上下文输入和上下文输出结果。此外一般情况下还会加入一个location embedding，从而使得非卷积的setup所忽略的位置信息也能够加入其中。  

而到了作者的模型中，输入是视频的特征表示和RPN产生的box proposal，将其映射到query和memory features，也就是key和value中，具体来讲，正在被分类的人是query，person周围的clip是memory，投射到key和value中。经过处理后，输出一个更新的query向量。作者使用了多个头，将每一层的多个头的输出连接起来，作为下一个层的query。  

key和value特征简单的通过trunk生成的特征图的线性映射来计算，输入的特征是T'×H'×W'×D。作者将center clip中person box的RoIPool-ed特征传入到一个query preprocessor(QPr)和一个线性层中，得到尺寸为1×1×D的query feature。QPr能够直接将特征平均，但是会失去所有人的spatial layout，所以作者首先使用1×1卷积进行降维，然后连接输出的特征图成一个向量，在最后再使用一个线性层降维到128D，和Q和K的尺寸相同。作者将这个QPr的变体称为HighRes，经过实验证明这种方式是效果最好的。  

剩下的结果严格符合transformer，用下面的公式进行:  
![55-1](/images/daily paper/55-1.png)  
作者使用了一个Dropout来处理得到的A<sup>(r)</sup>，然后将其加到原始的query特征中，得到的结果再传入一个残差分支中，包括一个LayerNorm和2层的MLP，以及一个dropout。最后的特征再通过一个LayerNorm，得到最终的query Q''，还是挺复杂的。下图详细的阐述了整个算法的流程:  
![55-2](/images/daily paper/55-2.png)  

### I3D Head  

为了衡量作者的Action Transformer收集的上下文的重要性，作者搞了一个简单的没有提取上下文的head架构。作者提取了一个与RPN提案相关的特征表示，RPN提案是使用时空RoIPool操作从特征图中得到的。详细流程见下图:  
![55-3](/images/daily paper/55-3.png)  

## Experiments  

训练细节、消融实验、可视化结果略过不表。和之前的STOA的比较如下图所示:  
![55-4](/images/daily paper/55-4.png)  
这里作者没有使用光流，而是只采用RGB图像作为输入，得到的结果却超过了其他方法，达到了STOA，结果还是不错的。  

## Conclusion  

这篇paper主要将Transformer引到了动作识别领域，然后得到了STOA的表现。不过这篇paper与小样本的关系并不大，并且主要聚焦在定位和复杂的动作识别，主要借鉴一下Transformer的使用方法。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

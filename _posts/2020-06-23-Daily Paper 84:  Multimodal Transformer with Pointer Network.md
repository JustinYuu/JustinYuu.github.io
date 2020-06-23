---
layout: post
title: "Daily Paper 84: Multimodal Transformer with Pointer Network"
description: "Notes"
categories: [MMML-Transformer]
tags: [Paper]
redirect_from:
  - /2020/06/23/
---

# Daily Paper 84: Multimodal Transformer with Pointer Network for the DSTC8 AVSD Challenge AAAI2020 Workshop  

## Introduction  

这篇文章和上一篇一样，研究的都是AAAI2020 Workshop中的AVSD-DSTC8任务。这篇文章使用multimodal transformer来将文本和视频的非文本特征结合起来，并使用pointer network来提升了对话文本生成的稳定性。该模型在automatic metrics上获得了很好的表现，在人工评价指标上获得了第五名和第六名。模型的效果可能没有上一个任务好，但是这篇文章的multimodal transformer可以借鉴一下。  

## Method  

对于AVSD任务上一篇已经讲得很明白了，这里直接讲模型好了。对于视频特征，作者所使用了在Kinetics Human Action Video数据集上预训练的ResNext101进行特征提取。对于音频特征，作者使用了预训练的VGGish模型来提取。提取的音频特征是128维，视频特征是2048维。  

作者使用了ACL19的Multimodal Transformer Network(MTN)作为骨架网络，MTN使用多头点积注意力机制来获取文本序列和视频特征的时序变化。整个网络采用Encoder-decoder架构，也采用了Transformer的LayerNorm。我不太懂为什么作者为什么不放流程图，只是用文字和公式进行网络模型的阐述，那我也只能尽可能的从上到下的用文字来描述作者的网络。  

首先看一下所谓的多模态注意力机制，我们知道多头注意力机制有QKV，这里是用另一个模态的信息作为Q，与本模态的KV做自注意力。  

接下来看一下整个网络的流程。首先把对话的query特征与时序视觉特征做多模态注意力，公式如下:  
![84-1](/images/daily paper/84-1.png)  

可见是由一套Transformer的encoder完成的，第一层是SA，第二层是CMA。  

然后把query特征和音频序列也做相同的多模态注意力，也是SA+CMA，公式如下:  
![84-2](/images/daily paper/84-2.png)  

在得到query-guided音频和视频特征之后，将它们通往decoder中，对文本输入和视频输入进行处理。具体来讲，对对话的目标回答Y进行embedding，然后通往4个text-to-text attention中，分别是自注意力，response-to-dialogue-history attention，response-to-query-attention和response-to-caption attention。公式如下:  
![84-3](/images/daily paper/84-3.png)  

最终的输出通过response-to-query的结果和query-to-audio的结果进行attenetion得到response-to-audio attention，然后用response-to-audio attention和query-to-vision attention进行attention得到response-to-visual attention，经过这一大堆操作之后得到最终的输出，公式如下:  
![84-4](/images/daily paper/84-4.png)  

我不太懂作者是怎么试出来的。。这一大堆复杂的attention操作看得我头晕，不过重点不在于该任务具体怎么搞的，而在于这个attention本身，所以看看就好了。  

此外还有一个比较重要的东西，叫做pointer network。这个网络是由Vinyals等人在2015年提出来的，作者使用pointer network来从不同的输入文本序列中选出tokens，并建立不同的词汇分布。具体来讲，给定输入文本X，得到embedding后的表示Z<sub>X</sub>和decoder最后一个注意力层的输出Z<sub>out</sub><sup>dec</sup>，可以通过点乘注意力建立一个pointer分布，公式如下:  
![84-5](/images/daily paper/84-5.png)  

对于回答的每一个词汇位置，都使用pointer分布来建立一个在词汇表上的分布，每一个token的概率都进行累积，得到最终的概率分布。给定回答的一个具体位置i，该位置的词汇分布由pointer分布定义的方式如下:  
![84-6](/images/daily paper/84-6.png)  

除了P<sup>X</sup>,作者还对history dialogue，caption和query分别求了pointer distribution，并使用一个线性变换+Softmax来使得模型能够生成不包含在任何文本输入序列的tokens，公式如下:  
![84-7](/images/daily paper/84-7.png)  

最后把这些concat起来，得到最终的contextual向量Z<sub>ctx</sub>，再乘以一个变换矩阵，经过Softmax得到最后的输出。  

值得注意的是，loss分为三部分，第一部分就是普通的负对数损失函数，第二部分是一个auto-encoding loss，该loss来源于MTN这篇文章，公式如下:  
![84-8](/images/daily paper/84-8.png)  

最终使用参数α和β(和为1)来控制比例，将三个loss加在一起得到最终的loss进行联合训练。  

此外作者进行了共同训练，将不同的视频特征类型和不同的特征与训练模型合在一起训练，对于每一个模型m，都能得到一个词汇分布P<sub>vocab</sub><sup>(m)</sup>，将这些结果加和之后再均一化，就得到最终的结果。  

## Experiment  

结果如下:  
![84-9](/images/daily paper/84-9.png)  

## Conclusion  

这篇文章是一篇应用文，我最想看的Multimodal Transformer没有介绍，只是把作者在参加比赛时的具体设置以及大量的trick详细的介绍了一遍。网络部分还要再读一下MTN这篇文章，这篇文章的trick可以记录一下，以后打比赛的时候可能会用到。和上篇文章的trick整合一下，目前有了以下一些trick：使用multi-task，以类似self-supervised的方式进行训练；使用联合loss进行训练；使用ensemble models采用不同的模型进行训练再取平均值；使用pointer network改善generator。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

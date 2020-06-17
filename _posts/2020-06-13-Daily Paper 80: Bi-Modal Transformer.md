---
layout: post
title: "Daily Paper 80: Bi-Modal Transformer"
description: "Notes"
categories: [CV-Attention]
tags: [Paper]
redirect_from:
  - /2020/06/13/
---

# Daily Paper 80: A Better Use of Audio-Visual Cues: Dense Video Captioning with Bi-modal Transformer  

## Introduction  

这篇是坦佩雷理工大学于五月下旬挂在arxiv上的，看格式投的应该是WACV或者BMVC。这篇文章主要使用了一个跨模态的transformer来做Dense Video Captioning任务。作者认为目前的Dense Video Captioning任务的主流方法大多只利用了视觉模态的信息，而对于听觉部分几乎完全忽略了。而作者提出了一个双模态的Transformer，使用seq2seq的方式来处理任意模态的输入。作者发现预训练的双模态encoder-decoder架构可以作为一个简单提案生成模块的特征提取器。该模块的表现在ActivityNet Captions数据集上得到了很好的表现。  

Dense video captioning我认为是比audio-visual event localization更为复杂的任务，它不但需要对每个事件进行定位，还需要产生一句话对齐进行解释。该任务一般是看成seq2seq任务来处理，因为输入的是视频序列，而输出是字幕序列。该任务的数据集是ActivityNet Captions,衡量指标是BLEU@3-4和F1指数。  

## Method  

整体的流程图如下图:  
![80-1](/images/daily paper/80-1.png)  

整个算法由两部分组成：双模态transformer和多头提案生成器。首先用VGGish处理视频的音频流，用I3D处理视频的视觉流，用GloVe处理Caption，然后使用N层的双模态encoder来处理音频和视觉特征，然后使用generated proposals对输入的特征进行修剪，根据预先定义的提案数选出指定数量的提案特征，然后将修剪后的特征再次传入encoder中。decoder关注双模态encoder产生的表示以及先前的caption word，并生成其上下文的内部表示，该内部表示被传递给生成器（右侧）以生成下一个单词。  

双模态表示在双模态decoder的每一层都使用，具体来说，encoder生成的表示传入到decoder后，优化了之前生成的caption。decoder的表示接下来导入到generator中，生成caption word。设计decoder的目的是输入一个embedded word的序列，因此，需要一个特别的初始token来生成第一个caption word，整个caption一个单词接一个单词的生成，直到一个特定的ending token发生。  

### Captioning Module  

Captioning Module的流程如下：首先使用双模态encoder来处理音频A和视频V特征序列，得到两个输出序列：audio-attended visual features和visual-attended audio features。接下来使用双模态decoder来处理这些features，双模态decoder将之前产生的caption word序列和这两个features作为输入，得到一个decoder输出，最终的decoder输出一个接下来caption words的词汇表概率分布，从而得到最有可能符合的下一个caption wordks。  

首先看下双模态encoder，encoder使用Self-Attention+Cross-Modal Attention+双层FC的架构，注意力使用multihead attention，具体方式如下图:  
![80-2](/images/daily paper/80-2.png)  

其次看双模态decoder，decoder和transformer略有不同，分为四部分：self-attention, bi-modal encoder-decoder attention, bridge和position-wise fully-connected layers，输出一个caption features，具体公式如下图：  
![80-3](/images/daily paper/80-3.png)  

最后是generator，generator的作用是根据bi-modal decoder输出的captiono features对下一个caption word的分布进行建模，说白了就是一个分类层，这很自然的可以想到用FC+Softmax来实现。  

Transformer使用了residual connection架构，也就是Add & Norm。作者还在五个地方使用了dropout，分别是在给原输出增加residual之前，在bridge layer的激活函数前，在positional encoding的输出去上，在position-wise FC层之间，以及在scaled dot-product attention的softmax操作后。  

### Event Proposal Generation Module  

该模块的作用是生成一系列generator，该模块由两部分组成，分别是双模态encoder和双模态多头proposal generator(与多头注意力机制无关)。双模态encoder输入了整个视频和音频特征序列，而上一个模块中的bi-modal只输入了与proposal有关的特征序列。由于视频和音频的维度可能不同，prediction的融合不可能在每一个时间步上实现，因此作者设置了多个timestamp，在每一个timestamp上独立的进行预测，从而组成一个跨模态预测的common pool，最后再从common pool中排序和选择最有可能的选项。具体的流程图见下图：  
![80-4](/images/daily paper/80-4.png)  

接下来看细节，首先是proposal generation head，该部分的灵感来源于YOLO，这部分其实是一个全卷积网络，这里和YOLO不同之处在于使用单一尺度进行prediction，而使用一个在每一个head内部不同的卷积核尺寸k来控制卷积的感受野大小，第一层的卷积层尺寸k，第二三层的卷积层尺寸为1。此外在所有层中使用padding和identity stride以保留原有的序列长度。  

其次是预测的方式，这里使用三个值来共同预测，第一部分是相对于一个位置p的片段中心点的分布σ(c)，一个用于锚框的系数exp(l)，和一个目标分数σ(o)，公式如下：  
![80-5](/images/daily paper/80-5.png)  

最终的预测输出是以grid-cell的形式，之后乘以与特征的时序尺寸相关的cell size以得到最终的时刻位置。  

每一个proposal generation head都会产生一个prediction，多个head就会生成一个common pool，最后基于common pool中的置信度取前100个进行处理。  

### Trainig Procedure  

简单介绍一下训练的细节。整个训练过程分为两步，首先使用ground truth proposal训练captioning module，然后使用预训练的双模态encoder来训练proposal generator。说白了就是整体很难训练，就分成两部分分别训练。作者使用了KL散度作为loss，使用了Transformer中也使用的Label Smoothing，使用了masking来忽略padding并防止模型提前接触到ground truth caption中的下一位置。训练完毕event proposal generation模块之后，每个模态中所有的proposal generation head都同时进行训练，然后将所以模态的所有head的loss值加在一起。在每一个head中，使用MSE进行定位的loss，使用交叉熵作为objectness losses。  

## Experiment  

简单放一下结果吧，结果如下图:  
![80-6](/images/daily paper/80-6.png)  

从上图可以看出该算法实现了SOTA。还进行了ablation study，结果如下图:  
![80-7](/images/daily paper/80-7.png)  

这里测试了不同模态的表现，以及不同的训练方式带来的差距，其实我蛮想看看只有transformer，没有proposal的部分效果如何的，没放应该是效果不咋样。  

## Conclusion  

总结一下，作者搞了一个双模态transformer，在ActivityNet Caption数据集上实现了SOTA。这篇文章做的东西和我做的其实挺像的，这也验证了这种encoder架构其实是有效的。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

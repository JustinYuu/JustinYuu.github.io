---
layout: post
title: "Daily Paper 59: DAM"
description: "Notes"
categories: [MMML-Event_Localization]
tags: [Paper]
redirect_from:
  - /2020/04/29/
---

# Daily Paper 59: Dual Attention Matching for AudioVisual Event Localization  

## Introduction  

这篇是audio-visual event localization最优秀的强化版本，和上一篇的ICASSP同一时期发出来，质量却有天壤之别。这篇paper最大的特色是使用了一个Dual Attention Matching双注意力匹配模块去处理视听事件定位问题。作者的主要思想在于同时捕获整体的高级别事件和局部时序的信息，从而得到更为广泛和准确的结果。这种方式可以同时应用在监督视听定位、跨模态定位等一系列audio-visual event localization任务上，并在AVE数据集上得到了SOTA的效果。  

作者回顾了之前的ECCV18和ICASSP19的两篇文章，指出这两篇文章的最大问题在于只聚焦了segment-wise的信息，而没有注意到两个模态之间的global temporal co-occurrences。所谓的global temporal co-occurrences，指的是在一段较长的时间内视觉和声音两个模态的相关性。作者发现一个事件经常会伴随着某一特定的声学事件和视觉事件，因此研究整体的多模态之间的相关性会显得更有意思。  

作者的贡献如下：提出了一个DAM模块，并在cross-modality localization task上实现了SOTA；在DAM的基础上设计了一个新的联合训练框架，在supervised audio-visual event localization上取得了优秀的表现。  

## Methodology  

整个算法的核心就是Dual attention matching DAM模块，该模块的流程图如下所示:  
![59-1](/images/daily paper/59-1.png)  

作者仍然使用了预训练的CNN进行特征提取。接下来的重点是将feature变成了local features和global feature两部分，对于每一个stream，都由当前stream的global feature和另一个stream的local feature进行element-wise match，也就是一个attention，具体形式是用内积来表示其跨模态的相似度，相乘后的结果是二元的标签，如果是event-relevant label会得到1，反之则得到0。  

这篇文章的思路其实很简单，就是把AVE-ECCV18的那篇注意力机制扩充到两个模态上了，之前的那篇是用audio-guided visual attention，仅用音频辅助的视觉attention，而这篇不仅使用了音频辅助的视觉attention，还使用了视觉辅助的音频attention。此外attention的内容也有所不同，这篇是用全局的当前模态和局部的另一模态进行attention，而AVE-ECCV18的attention是用的两个全局模态进行attention。  

再讲一下实现的细节，DAM模块主要由两部分组成：event-based sequence embedding和dual matching mechanism。首先来讲第一部分，对于global feature，其实是通过local features → event-relevent features → global feature转化而成的，作者在建立全局特征的时候去掉了背景片段以达到降噪的目的，而其event-relevant的实现是基于自注意力机制。首先从整个local feature中找到event-relevant的视频序列T<sub>E</sub>，将背景片段去掉之后，拼接得到event-relevant feature F<sub>E</sub>，然后用自注意力机制处理event-relevant feature，再进行时序平均池化，得到最后的event-relevant global representation。  

接下来是DAM中的dual matching mechanism。所谓的跨模态匹配，本质上是基于event segments和background segments所提供的信息不同而得出的。所以作者用另一个模态的local features与当前模态的event-based global features进行点乘，得到该模态当前事件的的features与另一个模态的所有时间段的features的相似度，从而找到相似度最大的部分，即为最为相关的结果。作者使用的是双模态交互，那么最终会产生两个预测结果，然后再取个平均即为最终的预测结果。整个DAM模块自然使用的是二元交叉熵进行优化。  

DAM模块可以应用到跨模态定位任务和全监督视听事件定位任务中。对于跨模态定位(CML)任务，不管是A2V还是V2A，DAM模块都可以直接应用上去，毕竟DAM的主要思想就是依靠跨模态检索和对比得出的。对于全监督视听事件定位(SEL)任务，需要略作修改，整体流程图如下所示:  
![59-2](/images/daily paper/59-2.png)  

如上图所示，最终的预测值是由两方面的预测值event category prediction和event relation prediction组成的，对于event relation prediction的预测，是通过DAM来完成的，对于event category prediction的预测，是通过图片左侧的两个attention完成的。这么做是因为作者将该任务分成了两部分：预测整个序列的时间类别和区分未修剪事件视频中的背景片段。作者使用自注意力机制得到视频和音频的全局表示，这里输入自注意力机制的是所有的片段，包括背景片段，之后把两个全局特征融合一下，得到基于融合特征的事件类别预测y<sup>c</sup>。与此同时，DAM使用global features，然后将所有的segment-level的局部特征和其对比，得到一个事件相关性预测y<sub>t</sub>，用来进一步决定该片段是否是背景片段。这里local visual featrues借鉴AVE-ECCV18的做法，使用audio-guided visual attention。如果y<sub>t</sub><0.5，该片段会被预测为背景类，反之则按照y<sup>c</sup>的预测结果进行类别预测。在训练的时候，由于同时得到了两个label（类别和相关性label），所以整体的目标函数如下图所示：  
![59-3](/images/daily paper/59-3.png)  

其中Lc是y<sup>c</sup>的交叉熵误差，L<sub>t</sub><sup>r</sup>是事件相关性的二元交叉熵误差。  

## Experiment  

CML的结果如下图:  
![59-4](/images/daily paper/59-4.png)  

SEL的结果如下图:  
![59-5](/images/daily paper/59-5.png)  

消融实验的结果就不放了，放一下SEL任务中λ的作用，这也是我比较疑惑的一点，作者采用两个loss一起训练的方式，究竟是为了强行应用DAM，还是DAM真的在SEL任务中发挥了作用呢？  
![59-6](/images/daily paper/59-6.png)  

结果就很迷。。λ取0.5是最好的，但是0.9的时候效果是第二好的，而λ为0或者1的时候反而又是最差的，我也不知道这是怎么来的。  

## Conclusion  

总结一下，看了几篇audio-visual event localization的文章之后，我也发现了所谓的创新点，要么从SEL任务出发，或是从CML任务出发，改进的方式基本上大同小异，基本上都是增添和更换各种各样的注意力。我觉得AVE-ECCV18的那篇对于fusion的创新还是有点东西的，我感觉可以从multimodal fusion的角度上进行创新，这应该是不同于当前各种花里胡哨的注意力的搬迁的新思路。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

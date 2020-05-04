---
layout: post
title: "Daily Paper 61: CMAN"
description: "Notes"
categories: [MMML-Event_Localization]
tags: [Paper]
redirect_from:
  - /2020/04/30/
---

# Daily Paper 61: Cross-Modal Attention Network for Temporal Inconsistent Audio-Visual Event Localization  

## Introduction  

这篇是南京理工大学发表在AAAI2020上的一篇audio-visual event localization上的文章，其主要改进是换了一个attention，使算法能够有效的在两个模态时序不对应的情况下达到event localization的STOA表现。  

作者使用了seq2seq的方式，使用三个不同的注意力机制来挖掘event发生的位置、时间和模态，然后更注意这些关键位置的信息。换言之，作者提出的算法可以剔除与事件无关的信息，利用最为有效的信息来进行定位。作者将该模型在SEL,WSEL和CML三个任务上进行了测试，均取得了很好的表现。  

## Method  

![61-1](/images/daily paper/61-1.png)  

该模型还是挺直观的，用了3个attention。第一个spatial attention，其实就是audio-guided visual attention，借鉴的是AVE-ECCV18的attention，与其不同的是在该attention和LSTM之间还加了一个self-attention，这其实是借鉴了ICCV19的那篇，然后通过LSTM之后再进行一个adaptive attention，这里是将另一个模态的self attention和当前模态经过LSTM的结果进行attention，还是类似ICCV19那篇的local feature和global feature进行attention，最后进行fusion和预测。  

看完流程图我是不太明白这篇paper的创新点在哪里。接下来看细节的介绍。首先是spatial attention，作者还算有良心，提到了这个attention是借鉴AVE-ECCV18的，不过也提到了不同点，即得到attention权重w<sub>t</sub>的时候，首先使用没有非线性层的MLP进行降维，使a和v维度相同，然后并不使用AVE-ECCV18的MLP去得到最后的权重w<sub>t</sub>，而是直接使用归一化内积操作来进行，作者认为内积操作可以衡量视频和音频特征之间的余弦相似度。  

然后是self-attention module。作者的目的还是通过捕获global correlation来得到"when" to attend。在我看来这里的流程和ICCV2019的self-attention并无不同，通过一个global context-aware self-attention weight，得到每一个独立的片段对于当前event localization的贡献度。  

最后是cross-modal adaptive co-attention module。之前的global feature主要研究的是单模态内的整体关系，而这里研究的是不同模态之间的联系。首先使用LSTM来进行时序依赖性建模，不过这里有所不同的是借鉴了visual sentinel的思想，在LSTM中添加了一个叫做modality sentinel的向量，如下图所示:  
![61-2](/images/daily paper/61-2.png)  

这里还是挺复杂的，所以详细介绍一下。大家都知道正常的LSTM在每一个时间步后都会生成一个hidden state h<sub>t</sub>，这里的LSTM稍作修改，除了生成<sub>t</sub>之外还生成一个modality sentinel s<sub>t</sub>。即有公式 s<sub>t</sub>, h<sub>t</sub> = LSTM(v<sub>t</sub>, h<sub>t-1</sub>, m<sub>t-1</sub>)。  

而s<sub>t</sub>由下列公式计算得到：  
![61-3](/images/daily paper/61-3.png)  


基于sentinel vector，作者提出了一个adaptive co-attention模块，用来计算新的语义向量，具体方式如下图:  
![61-4](/images/daily paper/61-4.png)  

新的语义向量c<sub>t</sub>是sentinel vector和attended audio features的混合体，该语义向量的生成过程中会经历一个trade off，用一个参数β来决定要多大程度的参考另一个模态的信息，每一个时间步的β<sub>t</sub>都不同。而另一个模态的信息c<sub>t</sub><sup>a</sup>的计算公式如下：  
![61-5](/images/daily paper/61-5.png)  

其中g是注意力函数，A是每一个时间步的audio feature，将A和h<sub>t</sub><sup>v</sup>导入到一个单层神经网络和Softmax中，得到在T个音频片段中的注意力分布。z<sub>t</sub>代表softmax的结果，z<sub>t</sub>也等同于共注意力权重α<sub>t</sub>，z<sub>t</sub> = W<sub>h</sub><sup>T</sup>tanh(W<sub>a</sub>A + W<sub>g</sub>h<sub>t</sub>)，其中W<sub>a</sub>和W<sub>h</sub>和W<sub>g</sub>都是需要训练的权重。  

为了计算β<sub>t</sub>，作者给α<sub>t</sub>新加了一个元素z<sub>t</sub>，该元素用来衡量该网络放了多少attention在sentinel上面。新的α<sub>t</sub>可以用以下公式计算：  
![61-6](/images/daily paper/61-6.png)  

;代表concatenation，最后的α<sub>t</sub>是整个sentinel vector和另一个模态的特征结合的注意力表示，β<sub>t</sub> = α<sub>t</sub>\[T+1]。最后把c<sub>t</sub>和h<sub>t</sub>合在一起，就得到了最后的跨模态视觉表示h<sub>t</sub>。  

之前的都是视觉模态的，对于听觉模态的完全相同，最后把h<sub>t</sub><sup>a</sup>和h<sub>t</sub><sup>v</sup>合在一起，就得到了最终的event localization。  

总结一下，这个cross-modal adaptive co-attention module主要利用了一个参数β，让算法自己决定采取哪个模态的信息，这一点倒是挺新颖的。作者认为生成的语义向量可以看做是当前隐藏状态的残差信息，从而减少了不确定性，并将当前隐藏层的信息作为互补信息考虑在内。    

## Experiment  

对于SEL和WSEL任务，该算法的准确率在使用ResNet-151提取的情况下可以达到77.1/75.7，还是挺不错的。  

## Conclusion  

总体来说，作者提出了三个attention，第一个attention几乎是照搬AVE-ECCV18的audio-guided visual attention，第二个attention和ICCV2019的SEL任务的左半部分非常类似，只有第三个attention我认为是真正有创新点的部分。同样是注意力，但是作者将重点放在了算法对不同模态信息的侧重和偏向性上，这在之前的研究中是未曾涉及的。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

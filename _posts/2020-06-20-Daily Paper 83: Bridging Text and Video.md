---
layout: post
title: "Daily Paper 83: Bridgeing Text and Video"
description: "Notes"
categories: [MMML-Transformer]
tags: [Paper]
redirect_from:
  - /2020/06/19/
---

# Daily Paper 83: Bridging Text and Video: A Universal Multimodal Transformer for Video-Audio Scene-Aware Dialog  

## Introduction  

这篇是北大和中科院发在AAAI2020 DSTC8 Workshop上的，也是个multimodal transformer的文章，这次的任务是Audio-Visual Scene-Aware Dialog任务，具体来说就是给定一个视频，机器要能够不断的回答提问者的问题从而和他聊下去。。。我不太懂为啥会有这么多奇奇怪怪的任务，反正这个任务是一个多模态任务就够了，那就能用multimodal transformer。AAAI2020针对该任务搞了一个竞赛叫做8<sub>th</sub> Dialog System Technology Challenge(DSTC8)，然后作者的算法在多个衡量指标上都得到了最好的表现。  

首先放一张图，看看啥是AVSD任务:  
![83-1](/images/daily paper/83-1.png)  

看起来和VQA差不多，区别在于每一个dialogue都有音频、视频、视频caption和dialogue summary，以及有10段对话。作者认为当前的主流操作是独立的使用encoder对不同模态的信息独立的编码，然后利用注意力机制来将不同模态的表示fusion起来，也就是主流的N-stream网络。作者认为这种方式并不能学习到最佳的多模态联合表示，因此采取了一个全局的多模态transformer来对不同的模态进行联合编码，并与此同时生成response。该任务输入的是多段序列，输出的也是一段序列，所以可以用encoder-decoder架构来完成。作者的模型基于预训练的GPT2模型调优实现，主要的contribution如下：第一个使用预训练的自然语言生成模型来处理多模态对话生成任务；将多模态特征整合到一个encoder中，并提出了一个multi-task学习方法去学习更好的联合表示并生成信息更丰富的回答；在AVSD数据集上实现了SOTA，遥遥领先了DSTC8-AVSD任务的其他队伍。  

## Method  

### Task Formulation  

首先介绍一下该任务以及一些标注。作者的目标是整合多模态信息，包括视频、音频和对话文本来生成富含信息并且流畅的问题回答。V代表视频，A代表音频，由于summary和caption有诸多相似，作者将这两者concat起来生成一个大的caption C，那么假设该对话有N对问答，那么用U={Q<sub>1</sub>,R<sub>1</sub>,Q<sub>2</sub>,R<sub>2</sub>, ..., Q<sub>N</sub>, R<sub>N</sub>}来表示整个问答文本，Q代表问题，R代表回答，其中R也可以分为m个r<sub>nm</sub>，代表m个单词。  

那么搞定了标注，该任务的目的就是给定Q, V, C和历史dialogue U<sub>＜n</sub>，回答给定的问题Q<sub>n</sub>，给出最优秀的答复R<sub>n</sub>的每个单词的概率，用公式表达如下:  
![83-2](/images/daily paper/83-2.png)  

### Model Overview  

模型示意图如下:  
![83-3](/images/daily paper/83-3.png)  

从上图可以看出整个模型是一个基于GPT2架构的多层Transformer encoder，更详细点来说，其实是一个12层的只有decoder的transformer。对于文本的输入，使用GPT2的设置，将输入的语句导入WordPieces中进行标注。对于视频和音频输入，将视频分为T个片段，每个片段有一个带有l个视频帧的滑动窗口，接着使用预训练的I3D-rgb和I3D-flow模型来提取rgb和光流的视频特征。由于音频和视频是对齐的，因此选用了相同segment的音频，使用预训练的VGGish模型来提取d维视频特征，接着将rgb、光流和音频特征concat到一起，得到video-audio特征VA。VA被通往一个FC层中(Video Embedder)，投射到和文本embedding相同的embedding空间中。  

正如上图所示，为了使得模型能够分别输入特征的不同模态部分，以及充分利用语序的顺序，每一个word token的最终表示是通过将每个单词的word embedding(WE), positional encoding(PE)和segment embedding(SE)加和得到。  

### Multi-task Learning  

为了调优该模型，作者引入了三个不同的任务，分别是根据视频、音频、caption和对话历史的Response Language Modeling，根据caption和对话的Video-Audio Sequence Modeling，根据视频和音频的Caption Language Modeling，看起来和自监督的trick类似，把这几个已知的信息分别去掉几个当成新的任务。  

首先是RLM任务，该任务使用负对数似然函数来实现，公式如下:  
![83-4](/images/daily paper/83-4.png)  

其中θ是可训练的参数。  

其次是VASM任务，作者使用video-audio特征回归方法来预测，使用L2loss来训练该任务，公式如下:  
![83-5](/images/daily paper/83-5.png)  

最后是CLM人为奴，和RLM任务类似，也是负对数似然估计，公式如下:  
![83-6](/images/daily paper/83-6.png)  

## Experiment  

实验的baseline有三个，首先是naive fusion，由任务的发起人提供，使用一个投影矩阵来整合所有的模态信息;其次是层级注意力Hierarchical Attention;最后是MTN Multimodal Transformer Networks，该文章接下来会介绍到。  

实验的metric有两个，第一个是objective evaluation，即自然语言生成实验中普遍采用的衡量方式，比如BLEU，METEOR，ROUGE-L和CIDEr。第二个是subjective evaluation，对于对话生成任务来说，该评价指标是很重要的，该任务的组织者找了很多人来根据回答的正确性、通顺性、信息含量和恰当性来给出他们主观的评价。  

评价结果如下:  
![83-7](/images/daily paper/83-7.png)  

## Conclusion  

作者提出了一个基于预训练模型的多模态的对话生成模型，并使用multi-task learning的方式来学习到更为准确、更富有信息的的表示。作者认为可以使用更多视频特征，比如ResNet特征来改善性能，也可以该方式应用到其他多模态任务中。在我看来，这篇文章的transformer显得中规中矩，并不是我理解中的multimodal transformer，而是将多模态信息简单的concat到一起之后生成联合表示，然后通往transformer中。不过作者提出的multi-task倒是挺有意思，我觉得和self-supervised有异曲同工之妙，之后有机会可以试试这种实验方法。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

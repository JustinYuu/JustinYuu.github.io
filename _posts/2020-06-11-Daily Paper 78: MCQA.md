---
layout: post
title: "Daily Paper 78: MCQA"
description: "Notes"
categories: [CV-Attention]
tags: [Paper]
redirect_from:
  - /2020/06/11/
---

# Daily Paper 78: MCQA: Multimodal Co-attention Based Network for Question Answering    

## Introduction  

这几天被程序搞的焦头烂额，论文也没怎么看，daily paper都快变成weakly paper了，这两天多看几篇。  

这篇文章是马里兰的Abhishek Kumar于四月底挂在arxiv上的，主要提出了一个基于多模态共注意力的网络，用于Question Answering问答任务。MCQA显式的融合并对齐了多模态的输入，比如文本、声音和视频等等，这些多模态的输入组成了query的内容。MCQA不仅在这个语义内部融合和对齐了问题和回答，还使用了共注意力机制来实现了跨模态对齐和多模态context-query的对齐。作者在Social-IQ数据集上评估了MCQA的表现，发现该算法提升了4-7%的性能。  

Social-IQ数据集貌似是一个VQA任务的数据集，含有7500个问题，52500个回答，1250个视频，人类的表现是95.08%，而目前最好的机器表现是64.82%。而作者最主要的contribution就是在Social-IQ数据集上取得了SOTA的表现。具体而言有三点，首先提出了MCQA，该算法包含两个模块，多模态融合和对齐模块，以及多模态Cotext-Query对齐模块，前者把三种模态输入融合对齐成为针对query的一个context，后者进行跨模态的对齐；其次使用了Co-Attention来实现对齐，这一方法借鉴的是Machine Reading Comprehension的文章；最后MCQA实现了Social-IQ数据集上的SOTA。  

## Method  

看起来网络并不复杂，流程图如下：  
![78-1](/images/daily paper/78-1.png)  

输入的数据有五部分，分别是文本x<sub>t</sub>, 音频x<sub>a</sub>，视频x<sub>v</sub>，query q和 answer c。原始的输入特征采用Social-IQ数据集released的版本，分别使用BERT,COVAREP和预训练的DenseNet161网络进行特征提取，使用CMU-SDK库来自动对齐三个模态。之后，作者进行多模态融合和对齐来获得输入query的语义信息，之后使用多模态Context-Query对齐来获得预测的结果h。  

从流程图中我们可以发现两部分都大量的使用了Co-Attention机制，这一机制来源于机器阅读理解领域的一篇paper，co-attentioin组件计算了两个模态输入的浅层语义相似度，这是通过构建一个软对齐矩阵S实现的。这里的输入是模态信息经过BiLSTM后的输出值，每一个矩阵S的元素入口都经过一个ReLU计算，公式如下:  
![78-2](/images/daily paper/78-2.png)  

作者使用矩阵S的注意力权重来获得向量up和uq的优化值，优化向量up和uq其实就是两者关联最密切的部分，公式如下:  
![78-3](/images/daily paper/78-3.png)  

这个attention应该是一个soft-attention，为啥不用tanh而用ReLU我也不太明白，可能是实验效果比较好吧。最后得到了一个u<sub>pq</sub>，就是把两个向量concat起来，就成为了u<sub>p</sub>和u<sub>q</sub>之间soft-alignment的捕获结果。那么把三个模态的信息分别导入BiLSTM中，然后将p和q分别赋值为x,t,v，就能得到u<sub>ta</sub>,u<sub>vt</sub>,u<sub>av</sub>三个多模态语义表示。然后再将这六个concat在一起，得到最终的多模态语义表示，公式如下：  

![78-4](/images/daily paper/78-4.png)  

之后就进入了第二个模块，即Multimodal context-Query alignment，把得到的多模态语义表示和query对齐到一起，这里还是用的co-attention，分别把u和q，u和c处理一遍，得到v<sub>uq</sub>, v<sub>uc</sub>，然后再将这五个concat起来导入Bi-LSTM中得到结果h，再经过一个线性自对齐组件得到最终的结果，公式如下:  

![78-5](/images/daily paper/78-5.png)  

## Experiments  

结果如下：  
![78-6](/images/daily paper/78-6.png)  

## Conclusion  

总结一下，作者提出了一个基于Co-attention的MCQA算法，在Social-QA数据集上实现了SOTA。这篇文章的VQA领域我是一窍不通，据作者的引用也可以看出idea几乎是照搬其他领域的attention，但是尽管如此，这个看起来简洁高效的Co-attention应该可以应用到音频和视频的多模态fusion当中。  
  
---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

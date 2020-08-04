---
layout: post
title: "Daily Paper 93: Synthesizer"
description: "Notes"
categories: [NLP-transformer]
tags: [Paper]
redirect_from:
  - /2020/07/29/
---

# Daily Paper 93: Synthesizer: Rethinking Self-Attention in Transformer Models  

## Introduction  

这篇文章是Google五月底挂在arxiv上的，主要研究的是对传统Transformer架构的剖析改进。我个人认为这篇文章做的东西还是相当有意思的。在传统的Transformer架构以及其一系列改进版本中，万变不离其宗的核心架构是点乘自注意力机制。但是作者在这篇文章中对点乘自注意力机制的必要性提出了疑问，并对于Transformer模型的点乘自注意力机制的作用和重要性进行了探究，通过实验，作者发现(1)随机对齐的矩阵作为注意力权重矩阵也能获得很好的表现，(2)通过token到token(也就是query-key)的互动其实并不重要。那么基于这个惊人的发现，作者就提出了Synthesizer，一个不需要token到token互动就可学习到注意力权重的模型。作者经过试验后发现Synthesizer模型能够在一系列任务中获得和原始transformer相当的表现。  

transformer架构是一个完全由自注意力机制组成的网络，大家普遍认为该模型之所以取得如此优秀的表现，正是由于其内部最重要的多头自注意力模块。但是作者在这里对点乘自注意力机制这一大家认为理所应当的重要组成部分进行了质疑：我们真的需要点乘自注意力吗？也就是说这篇论文的前半部分可以看做是一个ablation study，探究transformer中的点乘自注意力模块究竟发挥了多大的作用。  

从原理上看，点乘自注意力机制最大的作用就是学习自我对齐，通过对序列内某一个token和所有其他token以点乘的形式计算相似度，从而得到注意力权重，这也是之所以叫做点乘自注意力的原因。在计算相似度的时候，引入了query、key和value的概念，这其实是模仿了一种基于内容的检索过程，其核心是利用了pairwise interactions，这篇文章的主要目的就是rethink该过程的作用。那么基于该想法，作者就在想能不能不使用点乘自注意力，也不使用基于内容的类记忆的自注意力机制。在传统的注意力机制中，注意力权重是在实例或者样本阶段中通过pairwise interactions学习到的，这就会导致不同实例得到的注意力权重可能会根据每个实例各自的特征而波动，不利于学习到一个恒定的全局上下文。  

作者提出的Synthesizer架构不需要手动计算成对的点积，而是直接学习去合成一个自对齐矩阵。作者提出了一系列合成方法并进行实验衡量了结果表现，并发现在一些NLP的任务上能够获得和Transformer相当的表现。总结一下作者的contribution：提出了Synthetic Attention，在不需要显式的通过点乘注意力或者基于内容的注意力的条件下，通过生成与token到token依赖性无关的对齐矩阵来进行注意力操作，此外还实验了多种生成对齐矩阵的方式；作者还提出了Synthesizer，这是一个使用Synthetic Attention的新模型，能够在一系列NLP任务上得到和SOTA的Transformer模型相当的表现；此外，作者还证明了随机生成的对齐矩阵也能获得很好的表现，以及token到token的依赖性对于Transformer模型在一些任务上获得好的表现来说并不是必要条件。  

## Method  

接下来详细的看一下模型的架构。作者提出了两种Synthesizer。回想一下维度，输入的X维度是l×d，而输出Y的维应该也是l×d，其中l是序列的长度，d是特征维度。那么用一个矩阵来进行注意力操作的话，首先要使用一个投影函数F把输入X的维度变成l×l，也就是说对于序列里的每一个token，都把其特征维度从d变成l，得到的尺寸为l×l的矩阵B代表了每一个token对于其他token的权重，之后就可以用Softmax(B)G(X)来得到最终的Y，其中函数G()类似transformer中的V(value)函数。所以说这里的F函数其实是代替了Transformer中的QK<sup>T</sup>点乘操作。  

作者的两种Synthesizer也就是两种不同的F函数。第一种叫做Dense Synthesizer，使用两层FC和ReLU实现，公式如下:  
F(X) = W(σ<sub>R</sub>(W(X)+b))+b，其中σ代表ReLU。  

第二种叫做Random Synthesizer，如果说上面的Dense比较随意的使用两个FC处理的话，那么这个就随意的不能再随意了，作者只是随机了一个l×l的矩阵，将这个与token压根没有半毛钱关系的矩阵作为注意力权重……对于这一部分作者尝试了两种不同的策略，分别是将随机的权重值固定和训练。  

下图是两个synthesizer的架构图:  
![93-1](/images/daily paper/93-1.png)  

从参数数量上看，Dense Synthesizer添加了d×l个参数，而Random添加了l×l个参数，这里是忽略掉原始Transformer的QK矩阵而言的，所以实际上添加的参数没那么多。不过作者大方的不计较这些参数了，就看上述增加的参数而言，当l很大的时候计算会显得很繁琐。因此作者提出了synthesizer模型的因式分解，并证明了这些改进方式能够获得和原始状态相当的表现。  

作者认为分解后的输出不但能够少幅减少原始synthesizer的参数，还能够有效的避免过拟合。对于输入矩阵X<sub>i</sub>，分别用两个不同的投影函数将其投影成为矩阵A,B，公式为A,B=F<sub>A</sub>(X<sub>i</sub>),F<sub>B</sub>(X<sub>i</sub>),由于是矩阵的因式分解，所以A是a维，B是b维，a×b=l。接下来有C = H<sub>A</sub>(A) * H<sub>B</sub>(B)，H函数是一个平铺函数，其作用是把某一个向量在单一维度复制k次，即从l变成lk，那么H<sub>A</sub>就是将a变成ab，H<sub>B</sub>将b变成ba，最后需要一个结果，所以将两个H函数的输出合在一起。  

之前说的是Dense Synthesizer，而Random可以使用类似的操作，用低秩的矩阵来代替原有的随机矩阵R，公式如下:  
![93-2](/images/daily paper/93-2.png)  

很容易可以看出，对于每个头，超参数都从l²变成了2(lk)，而k<<l，所以达到了参数的减少和避免过拟合。在实验中作者选取k=8。  

在最后的最后，作者认为这些提出的synthetic注意力都能够用加法的形式来合在一起，公式如下:  
![93-3](/images/daily paper/93-3.png)  

其中S函数是一个参数化的生成函数，α(∑α=1)是可学习的权重，对于Random和Dense的混合来说，应该长这个样子:  
![93-4](/images/daily paper/93-4.png)  

最后作者进行了一下总结，包括了两个synthesizer方法以及其因式分解形式的公式和参数等信息，值得一提的是原始Transformer也可以通过synthesizer来生成，只需要把F函数分别变成Q矩阵和K矩阵即可。总结的表格如下:  
![93-5](/images/daily paper/93-5.png)  

## Experiment  

实验部分就略微介绍一下。作者在多个任务上进行了实验，包括机器翻译、语言建模、文本生成、以及Multi-task nlp任务。  

在机器翻译、语言建模和文本生成任务上的结果如下图:  
![93-6](/images/daily paper/93-6.png)  

在Multi-task NLP上的结果如下图:  
![93-7](/images/daily paper/93-7.png)  

实验的结果还是挺有意思的。在单任务上用Transformer做baseline，结果显示机器翻译和语言建模任务上Random+Vanilla的表现居然是最好的，文本生成Dense在对话的表现上比较优秀，而在Summarization上大家都差不多。  

在多任务上用的baseline是T5，Random+Vanilla的表现非常优秀，在很多指标上超越了T5。这说明random作为T5的补强部分能够有效的提升模型的整体表现。作者还做实验研究了head数量的影响，结果显示h=32的时候效果最好。  

## Conclusion  

总结一下，作者提出了Synthesizer架构，不使用点乘自注意力，而是使用其他方式来获得与token-to-token无关的自注意力权重，得到了不逊于基于点乘自注意力的transformer表现。这对传统的transformer之所以有效的原因产生了新的剖析和结果，实践证明tokne-to-token的模式并不是transformer所需要的，那么是否有新的权重生成方式来提高transformer的性能？或者有没有更简洁的方式来提高其效率、减少参数数量？  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

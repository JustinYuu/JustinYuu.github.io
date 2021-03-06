---
layout: post
title: "Daily Paper 98: CCL"
description: "Notes"
categories: [Audio-Visual]
tags: [Paper]
redirect_from:
  - /2021/04/25/
---

# Daily Paper 98: Distilling Audio-Visual Knowledge by Compositional Contrastive Learning  

## Introduction  

这篇CVPR2021的文章所做的事情和我之前一直想要做的高度类似。被scoop的伤心之余，仔细研读一下这篇文章。  

在多模态的若干任务中，一直以来的方式是用单模态预训练网络提取特征，然后使用若干模态交互的方式来获取多模态特征。然而，近年来的知识蒸馏做到了从大模型中将知识蒸馏到小模型中。那么很自然的一个想法就是，能不能采用已经非常成熟的单模态方法作为teacher network，将多模态的网络作为student network，创立一种新的框架来完成跨模态的知识蒸馏呢？举个例子，以多模态的视听时序动作检测为例，如果能够将当前SOTA的单模态的视觉时序动作检测(TAL)和音频的事件检测方法(SED)作为教师网络，而将多模态的视听时序动作检测网络作为学生网络，能不能获得比当前多模态方法更好的表现呢？  

如果这一想法成为可能，那么等于说提出了一种全新的多模态学习的范式，也就是说，能使用一种更为新颖而有效的方式(跨模态知识蒸馏)来更好的利用单模态的先验知识，从而提高跨模态的任务表现。此篇文章和我的想法大致相同，只不过实现方式上更为泛化，并没有针对某一特殊的任务，而是通过跨模态的知识蒸馏训练了一个audio-visual representation，进一步应用到下游任务中。而这篇文章和之前的表示学习不同之处在于，虽然用于蒸馏的知识可能在语义层面上并无关联，但是作者提出的框架能够学习compositional embedding，有效的弥补语义层面上的差距，并捕捉与任务无关的语义信息。此外，这篇文章很大的贡献在于在三个数据集上创立了多模态蒸馏的benchmark，分别是动作识别的UCF101，时序动作检测的ActivityNet，以及多模态数据集VGGSound。  

## Methodology  

作者的核心内容，也是跨模态蒸馏的核心问题，就是如何设计蒸馏框架。作者设计了一套名字为Compositional Contrastive Learning(CCL)的方案，该方案能够从语义不同的多模态信息中获取并蒸馏跨模态的内容。具体来说，音频和图片先使用预训练的教师网络提取特征，然后与视频学生网络的embedding结合从而弥补语义差距，那么此时便有了音频特征，图片特征，视频特征和复合特征，这四个特征使用CCL的方法，进行音视频知识的蒸馏。在测试的时候，只有视频学生网络被应用于视频识别或者视频检索任务中。具体的流程如下图所示:  
![98-1](/images/daily paper/98-1.png)  

整个框架分为几部分。首先是单模态部分，对于音频特征，使用1D的CNN作为教师网络，图像教师网络是标准的2D CNN，图像是从视频帧中随机抽取的，视频学生网络使用3D的CNN来提取，使用交叉熵误差来预测视频类别。而由于学生和教师的embedding可能存在语义的不对齐，所以需要建立一个复合的多模态表示。具体来说，教师特征和学生特征以残差的形式进行融合，如下图所示:  
![98-2](/images/daily paper/98-2.png)  

其中F函数是可学习的复合函数，f函数是多模态的融合函数，由归一化，concat和FC层组成。整个训练的过程要优化F函数，F函数使用视频的分类交叉熵损失优化，从而尽可能使得学生网络的分类结果和教师网络相同。训练完F函数后，即可得到音频视频、图像视频的复合特征，进一步使用交叉熵损失函数优化即可:  
![98-3](/images/daily paper/98-3.png)  

在之前的知识蒸馏中，大多只是对学生网络和教师网络的预测结果进行蒸馏，即尽可能的使学生网络的预测结果和教师网络的预测结果相同。然而由于多模态场景下预测的类别可能与单模态场景不同，这一策略无法在多模态蒸馏中使用。因此作者提出了在隐特征空间中进行度量学习，然后在预测空间中对比类别的分布。具体来讲，对于已经得到的单模态和多模态复合特征，作者尽可能的将同类别的模态对聚在一起，而将不同类别的模态对分开，其中4个模态(音频、视频、图像、复合)两两之间进行度量学习，各用一个度量损失函数进行优化，音频和视频对之间的优化函数如下图:  
![98-4](/images/daily paper/98-4.png)  

其中φ表示余弦相似度，τ是温度参数,而概率p表示在整个小批量的音频特征中，将该视频和对应的音频恰好匹配的概率。而由于对于一个batch中的第k个类别，可能存在多个这样的匹配对，所以将之前的损失函数换为多类别噪音对比估计函数(multi-class NCE loss)，公式如下:  
![98-5](/images/daily paper/98-5.png)  

Bp,Bn是正样本对和负样本对的数量，正样本对即属于第k类的样本，而负样本是不属于第k类的样本。用这种损失函数能够将类别之间的差异性和相似性也考虑在内，避免了将一些正样本误判为负样本，进一步使得网络对正样本能够分配更大的概率。该损失函数除了应用在单模态之间外，也可以应用在复合模态和多模态之间，从而进一步蒸馏多模态知识，公式如下:  
![98-6](/images/daily paper/98-6.png)  

用一个λ参数调节比例。以上的公式是音频和视频之间的蒸馏，对于图像和视频之间，也使用类似的方式，公式类似，因此在此略过。  

上述的公式其实是在特征空间内进行约束，然而除了在特征空间中，也可以在预测空间中进行约束。这里作者使用了Jenson-Shannon divergence(JSD)进行预测约束，公式如下:  
![98-7](/images/daily paper/98-7.png)  

那么最后，整个复合度量学习目标函数(CCL)就可以写成下述形式:  
![98-8](/images/daily paper/98-8.png)  

## Experiments  

作者在三个数据集上进行了实验，分别是UCF101,ActivityNet,VGGSound。除了VGGSound，其他两个数据集的音视频部分都是不关联的。视频学生网络使用的是R(2+1)D-18,音频和图像教师网络用的是AudioSet上预训练的1D-CNN14和ImageNet上预训练的2D-ResNet34。任务选取了视频分类和检索两个任务，但是训练只用了视频分类进行训练。分类使用top-1准确率作为指标，检索使用R\@K作为指标，检索的时候，使用测试集的视频作为query去检索训练集的视频。  

分类的结果如下图:  
![98-9](/images/daily paper/98-9.png)  

检索的结果如下图:  
![98-10](/images/daily paper/98-10.png)  


在VGGSound的结果如下图:  
![98-11](/images/daily paper/98-11.png)  

ablation study和分析就不放了。  

## Conclusion  

多模态蒸馏一直是一个可行的新领域。CVPR21上有几篇很不错的文章，这篇文章我认为做的应该是最好的。作者提出了一个看起来很复杂，其实结构很清晰的训练框架，也进行了非常充分的实验，并取得了不错的效果。我认为最为难得的是，这篇文章提出的方法克服了跨模态蒸馏最为困难的语义不对齐问题。在此之前，我一直认为只能用VGGSound这种音视频语义对齐的数据集作为跨模态蒸馏的数据集，但是这又大大降低了跨模态蒸馏的适用性，而CCL这种方式可以在任何存在多模态数据的数据集上进行应用，大大提高了泛化性能。基于此文章，对其他任务的跨模态蒸馏，对该任务跨模态蒸馏方法的改进(教师网络和学生网络的交互方式)，还有进一步的研究空间。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

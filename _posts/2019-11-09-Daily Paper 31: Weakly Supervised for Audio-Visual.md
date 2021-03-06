---
layout: post
title: "Daily Paper 31：Weakly Supervised for Audio-Visual"
description: "Notes"
categories: [MMML-Self_Supervised]
tags: [Paper]
redirect_from:
  - /2019/11/09/
---

# Daily Paper 31 - Weakly Supervised Representation Learning for Unsynchronized Audio-Visual Events  

## Introduction  

这篇paper是CVPR2018 Workshop上的文章，是由萨克雷和特艺合作的一篇paper，主要利用视频和音频两种模态，实现对视频中事件的分类和定位。整个网络是一个弱监督学习系统，仅仅使用视频级别的事件标注，没有标注任何时间信息，但是这个网络却可以从不同步的音频-视频事件中进行学习，并在一个大规模的弱标注音频事件的影响中得到了当今最好的表现。通过对定位的图像区域和音频片段可视化，可以看出该系统是高效的，特别是在处理嘈杂的情况的时候，因为这时与模态有关的暗示异步出现。  

作者之所以选择弱监督学习的方式是因为标注太麻烦了，因为作者想完成的任务是视频中的事件分类和特征性视听元素定位，那么要进行监督学习的话就要对每一个视频中的确定时间点进行事件标注，并且对每一个事件中的特征性元素进行标注，这将会花费大量的事件，并且还不一定准确，因此作者干脆根据整个视频的特征对视频本身进行分类标注，不对时序信息进行标注，让网络自凭本事学习特征。  

这里举了一个火车的例子来解释异步视听信息，比如在一个视频片段的前半部分，铁轨上没有火车，但是有火车的汽笛声，在视频的后半段没有汽笛声，但是有火车出现，那么这时候视听信息都含有火车的信息，但是并不是在同一个时间步出现的。因此要理解视频的语义，只通过同一个时间步的信息进行训练有可能会错过很多信息，因此作者这里采取异步训练的方式。  

该任务和之前AVC、AVTS等任务的区别在于之前只是简单的探究图像和音频的对应关系，但是这里需要学习用于分别真实世界中物体和对应的视听线索的数据表示。这一任务是之前没有人做过的，这本身也就是这篇paper的创新点。这一任务可以被称为Multiple Instance Learning（MIL），MIL问题可以解决那些从一系列例子中进行标注而不是对单一实例进行标注的任务，也就是说该任务可以从多个实例中寻找信息进行学习。这里多个实例就是视频中出现的多个图像区域和音频片段，我对这个任务的理解是需要把时序前后的信息记忆下来，所以我第一时间想到的是LSTM或者GRU。这里作者采取了不同的策略，首先将视觉和听觉部分分开处理，提取特征后计算他们与各个标签的相关性分数，汇总两个模态的分数后再结合起来进行视频级别的分类，学习到用于事件分类和定位的表示。  

作者的主要贡献如下：提出了一个视频分类和视听源定位的联合训练多模态框架，能够处理异步案例，并在视频分类中获得了当今最好的表现。  

## System  

整个系统的流程很简单，分别提取声学和视频特征，然后分别通入两个网络进行处理，接着每个子网络得到的结果又分别分为两部分：定位和分类，两部分各由一个独立的FC层处理，接着使用element-wise乘法将两部分结合在一起，然后求和并进行l2正则化，两个子网络得到的最终结果加起来成为最后的分类结果。  

首先来看特征提取，首先是视觉特征，我们都知道faster R-CNN提出的区域提案算法成为了当今目标检测的核心算法，这里也是使用的这一方法。对于提案的选择，由于作者需要对最具辨别力的区域进行时间和空间上的定位，所以作者选择在下采样的视频帧序列上应用区域提案算法。具体来讲，作者以1fps对每一个视频进行下采样，使用EdgeBoxes对下采样图像进行类不可知的区域提案生成，这种生成方法是考虑到一个box里面的coutour数量反映了一个物体存在的似然。EdgeBoxes这个算法对于每一个边框都额外的生成了一个置信度分数，它能够反映这个边框的“物体性”，即含有物体的可能性，作者使用这个置信度分数来选择每一个采样图片中的前M个提案，这样能够显著的降低计算复杂度和冗余。作者使用卷积网络和RoI池化层对每一个区域提案中的特征向量尽心给处理，在不同的区域内参数共享从而加速计算，经过RoI池化后的向量再进入两个fc层中进行调优，这里的CNN使用ImageNet预训练的初始化权重。  

接下来看声觉特征，作者首先将原始音频的波形图作为对数梅尔谱图，接下来通过一个沿着时序在谱图上滑动的固定窗口获得每一个提案，窗口的大小与音频特征提取器的尺寸相适应。作者使用一个VGG风格的网络作为音频的处理网络vggish，该网络已经在YouTube-8M上预训练过，堆叠了4个卷积层和2个fc层，生成了128维的embedding。  

最后就是将提取的特征导入提案分数网络中，再将其融合在一起。作者使用了two-stream双流模型，计算他们相对于每一个class的分数，在这个双流模型中，两个模态的网络架构是一模一样的，两个流分别负责分类和定位，不过在定位流中会加一个softmax来选择每一个class中相关性最高的提案，将定位流的输出和分类流的输出逐元素相乘，再讲所有区域的score加起来就得到了最终的class score。由于作者的分类中一个提案或者片段有可能同时属于多个class，所以没有对分类流添加softmax。在做完所有操作后，对全局的视频级别的分数进行l2归一化，将分数归一化到相同的范围内，作者证明这能够得到更好的表现。  

## Experiment  

作者使用了DCASE在大规模弱监督的为智能汽车设计的声音事件监测挑战赛的数据集，这是一个AudioSet的子集，包含大量的弱监督的无约束YouTube视频，视频内容大多是鸣笛声和车辆行驶的声音。由于之前并没有类似的研究，因此没有现成的baseline可供使用，作者只能自己设计了几个baseline进行对比：1.AV单流架构，使用一个stream来处理MIL问题，将max替换为log-sumexponential。2.两个单模态网络，以及一个他人开发的弱监督深度检测网络WSDDN。3.基于Gated Convolutional RNN的CVSSP Audio-Only网络，这个网络是DCASE 2017音频事件分类子任务的冠军，也是state-of-the-art方法。  

实现细节略。  

使用F1分数作为评价指标。结果显示作者的双流网络拥有最高的F1分数和最高的召回率，查准率排名第二，第一是CVSSP的Fusion系统，总体来看作者的模型还是超越了当时最好的模型的。作者还对每个类别的准确率进行了分析，为了方便了解和分析最终的结果，作者将类别分为两大类：有清晰的视听元素的类和一些警报声的类。最后显示视觉信息非常明显的视觉特征能够显著的提升通过声音单模态网络的表现，同时对于警告声的类别，只有视觉特征的网络表现是不够好的，还需要声觉信息进行补充，总之能够很好的证明声觉和听觉具有互补性。但是我个人觉得这种互补性体现的并不是很好，因为视觉信息非常明显的特征很明显要通过视觉网络得出，而警告声很明显也需要通过听觉特征分辨出，那么作者这种用不合适的模态得出较差的结果，加上比较对应的模态得到较好的结果，并不是很能得到互补性的结论，因为就作者举的例子而言，第一部分的声觉特征究竟起到了多大的帮助并没有说明，而第二部分的视觉特征究竟有没有起作用也是未知数。  

此外由于CVSSP方法在类的表现比作者的系统更好，作者认为这是由于他们音频系统的时间建模不够好所致，这里使用固定的粗略时间窗口960ms，可能并不完全适应所有的音频事件，作者认为RNN可能会更好一点。  

此外作者还进行了定性的评价，结果显示定位还是挺准确的，此外不同区域的区域提案heatmap和音频片段分数显示定位中存在着时间的不同步，不过最终的预测结果是正确的，这代表着该模型能够结合异步信息进行学习和分析预测，基本实现了作者预期的目标。  

## Conclusion  

总结一下，作者主要提出了一个弱监督的深度多模态模型，用于视听事件定位和分类，最终得到了超出当今最好表现的结果。结果证明了跨模态互补信息的重要性，并给出当今的一些单模态系统可行的发展方向。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

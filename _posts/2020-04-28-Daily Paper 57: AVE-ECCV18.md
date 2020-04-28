---
layout: post
title: "Daily Paper 57: AVE-ECCV18"
description: "Notes"
categories: [MMML-Event_Localization]
tags: [Paper]
redirect_from:
  - /2020/04/28/
---

# Daily Paper 56: Audio-Visual Event Localization in Unconstrained Videos  

## Introduction  

这个月写了一个月的代码，东西没搞出来，倒是开了个新坑：多模态的event localization方向，也看了几篇论文，这里把它整理一下。  

这一篇是Audio-Visual Event Localization方向的开山之作，是由Rochester的几位学者发在ECCV2018上的。Event Localization类似于时序动作检测，是将一段视频中的动作发生的时间段进行定位。作者收集了一个用于event localization的数据集AVE dataset，然后提出了三个任务：supervised audio-visual event localization，weakly-supervised audio-visual event localization，cross-modality localization。作者提出了一个audio-guided的视觉注意力机制，去探究audio-visual之间的联系，使用一个dual multimodal residual network(DMRN)去将两个模态的信息融合在一起，使用一个audio-visual距离学习网络去处理跨模态的定位，并获得了很好的效果。总体来讲作者的工作量还是挺大的。  

作者在提出了这三个任务的同时，也给出了这三个任务的basseline和他们的新方法。对于前两个任务，作者将序列标注问题作为baseline，使用CNN来将音频和视觉输入编码，使用LSTM来捕获时序依赖性，使用FC网络进行最后的预测。基于该baseline，作者提出了audio-guided视觉注意力机制来验证是否音频可以有助于视觉特征的提取，该方法同时证明了发声部位的空间定位也能用该方法得到。作者使用上面到的DMRN进行多模态的特征融合。对于弱监督学习，作者将其看做为一个Multiple Instance Learning(MIL)问题，然后通过增加一个MIL池化层来构建网络架构。对于跨模态定位任务，作者提出了一个audio-visual距离学习网络来测量任意视听对的相关性。  

## Dataset and Problems  

### Dataset  

AVE数据集是作者自己搞的一个数据集，因为这个任务是新的，没有对应的数据集进行训练，因此作者就从AudioSet选了一个子集，包括4143个视频，一共有28个事件类，每一个视频中至少含有一个时长为2s的audio-visual事件。事件类的类别跨度很大，每一个类别至少有60个视频，最多的类别有188个视频。  

### Fully and Weakly-Supervised Event Localization  

event localization的目标是预测每一个视频片段中的事件标签。具体来讲，给定一个视频序列，作者手动将其分为T个非重复片段，每一个片段时长为1s，对于每一个片段都有一个标注的类别y，y的类别包括所有AVE数据集的数量加上一个背景标签。对于监督event localization任务而言，事件的标签是对于每一个片段而言的，即1s一个标签。作者研究了单一模态的定位性能和多模态的定位性能，从而判断多模态的模型到底有没有作用。对于弱监督的event localization而言，作者只对整个视频的事件标注了一下，也就是说不是逐秒标注，而是视频整体标注一个tag，但是算法预测的时候仍然是逐秒预测，这种标注的原因是为了能够减轻模型对于精细标注的依赖性。。我觉得这个说的太牵强了，我感觉应该是为了提高模型的泛化性能，提高其应用能力，以及减轻数据集标注的难度。  

### Cross-Modality Localization  

跨模态定位是另一个不同的任务，该任务的目的是去找到某一个单模态的片段与之对应的另一个模态的片段。这就又分为A2V和V2A，即根据音频来找到视频和根据视频来找到音频两方面。该任务是event-agnostic的，也就是说不管什么样子的视频，不管有无标注的视频，都可以进行跨模态的定位。  

## Methods for Audio-Visual Event Localization  

该模型分为四个部分，整体框架、注意力机制、融合方法和弱监督扩展。  

对于整体框架，也就是作者提出的baseline，是当做一个序列标注问题来解决的。首先使用预训练的CNN进行音频特征和视频特征的提取。然后使用两个独立的LSTM来独立的进行两个模态的时序依赖性建模，输入为全局平均池化的视觉特征和音频特征，然后再用后续的fusion模块将两路的特征融合在一起，然后使用FC+Softmax进行逐片段的预测，使用一个multi-class交叉熵损失进行训练。对于作者的AVE网络，并不是直接将特征输入到LSTM中，而是加上一个audio-guided visual attention，将音频特征作为视频特征的辅助信息，生成一个跨模态的视觉特征。  

audio-guided visual attention其实很简单，就是将视觉特征和听觉特征搞到一个attention里面，得到一个注意力处理后的视觉特征。注意力的公式也很常规，就是用一个权重w去和t来相乘，权重来源于用一个多层MLP和Softmax去搞音频特征a<sub>t</sub>和视频特征v<sub>t</sub>。  

接下来是两路特征的融合。作者不是简单的融合，而是使用了一个叫做Dual Multimodal Residual Network(DMRN)的方式进行融合。这是借鉴了multimodal fusion中的Multimodal Residual Network的思想，该网络用语言表示起来太过抽象，直接上图比较好理解一点:  
![57-1](/images/daily paper/57-1.png)  

作者认为这种复杂的融合方式可以同时保留原模态中有效的信息和另一模态中的互补信息，我感觉和attention也有异曲同工之妙。其实也可以使用多层残差块的堆叠来学习一个深层的融合网络，但是作者实验后发现深层的MRN和DMRN都不会提升其性能，单层的残差块足以处理这种简单的融合任务了，深层的融合网络还会让网络变得更难训练。此外作者还做实验验证了在LSTM之后进行fusion的效果要优于在LSTM之前进行fusion，作者猜测这可能是因为LSTM之前的视频特征和音频特征没有时序对齐，所以需要用LSTM来对齐一下。  

对于弱监督event localization而言，作者将其视为MIL问题，因此将模型逐帧判断的标签进行MIL池化，将最后一个FC层的预测值取平均，再使用softmax得到最终的视频标签预测结果，从而进行训练。在测试的时候仍然预测逐帧的标签。  

## Method for Cross-Modality Localization  

为了解决跨模态定位问题，作者提出了一个audio-visual距离学习网络(AVDLN)，如下图所示：  
![57-2](/images/daily paper/57-2.png)  

整个网络还是比较简单的，主要来计算给定的V和A之间的距离，在测试的时候，对于A2V任务，作者使用了滑动窗口方法来优化下列目标函数: t<sub>\*</sub> = argmin∑<sub>s=1</sub><sup>t</sup>D<sub>θ</sub>(V<sub>s+t-1</sub>, A<sub>s</sub>)，t<sub>\*</sub>代表视频和音频内容开始对齐的时间点，t<sub>\*</sub>∈{1,...,T-l+1}，T是视频总时长，l是视频片段时长，该目标函数可以通过最小视频和音频片段的累计距离来计算最优的对应。同样的，对于V2A，也是相同的公式，作者直接省略了。  

确定了目标函数之后，还需要确定D究竟是什么形式，作者采用的是使用预训练的CNN来处理视频和音频，得到特征，然后使用两个不同的双层FC网络来降维，将两个FC网络得到的结果求欧氏距离，从而得到距离D。作者使用对比度损失来优化D。  

## Experiments  

简单说一下实验的结果，作者使用VGG-19的pool5特征层来提取提前选取的视频中的16个RGB帧，VGG19在ImageNet上进行预训练，接下来进行16帧的GAP，生成一个512×7×7的特征图。作者也尝试了C3D，不过并没有观察到明显的性能提升。作者使用一个在AudioSet上进行预训练的VGG-like网络上来对每一个1s的音频片段进行特征提取，得到128维的音频表示。  

作者将其V-att模型和A+V-att模型与V和A+V模型baseleine进行全监督和弱监督训练，观察和比较实验结果。此外还将其和当时的STOA时序标注方法ED-TCN和proposal-based SSN进行对比。作者将其融合方法DMRN和其他融合方法进行了对比，包括相加、GMB、GMU、Concat、MRN等等，并探究了early、late、decision fusion对性能的影响，这里early指的是在LSTM之前进行fusion，late是在LSTM之后，decision是在FC之后、Softmax之前。此外作者还搞了一个DMRN的进化版DMRFE，将视频和音频特征输入到两个分离的块中，然后使用average ensemble来结合两个预测的概率。  

对于event localization，作者使用整体准确率作为评价指标，而对于跨模态定位，作者的评价指标也很严苛，如果对应的部分和groundtruth完全一样，则视为成功，否则视为失败，计算所有样本的准确率。作者还将跨模态定位模型和DCCA方法进行了对比。  

对比的结果如下，首先是fusion模块的比较：  
![57-3](/images/daily paper/57-3.png)  

其次是event localization算法的性能比较：  
![57-4](/images/daily paper/57-4.png)  

最后是跨模态定位的性能比较:  
![57-5](/images/daily paper/57-5.png)  

## Conclusion  

总的来说，这篇paper的质量还是非常高的，做的东西也相当solid。简单总结一下，作者提出了三个新任务，一个数据集，以及针对三个新任务的两个算法，第一个算法是针对event localization的，其中使用了注意力机制和DMRN融合网络，第二个算法是用于跨模态定位的，使用的是metric learning学习的方式，做的效果也非常好。  

如果要说不足或者亟待改善的地方的话，那就是跨模态定位的方法还太过于简陋，应该有更多有效的新方法，以及event localization的注意力机制还不够彻底。一种很自然的改进方式就是用双重注意力机制，不只用audio来给visual加注意力，也用visual给audio加注意力。此外DMRN的融合方式很有可能会被更新的multimodal fusion方式超过，毕竟fusion也是一个研究成果更新很快的多模态研究方向。  

作者是将event localization和cross-modality localization两个任务结合在一起讨论的，在我的理解下这两个任务一个是multimodal多模态任务，一个是cross-modality跨模态任务，两个任务如果要进一步研究的话，应该会朝着两个不同的方向进行。event localization应该更多的讨论模态之间的融合方式和两个模态之间的结合方式（注意力），而跨模态任务更多是研究两个模态之间的相关性和不同点，使用类似metric learning的方式研究不同模态之间的距离变化，用一个更好的特征空间来进行时序上的比较，还是挺期待后续的研究进展吧。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

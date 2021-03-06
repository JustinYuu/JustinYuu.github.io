---
layout: post
title: "Daily Paper 23: Objetcs that Sound(AVE-Net)"
description: "Notes"
categories: [MMML-Self_Supervised]
tags: [Paper]
redirect_from:
  - /2019/11/01/
---

# Daily Paper 23 - Objects that Sound  

## Introduction  

这篇文章是DeepMind和VGG组合作的文章，主要解决AVC问题，这里VGG组之前提出了一个非常著名的L3网络，在下一篇文章中会进行介绍。  

这篇paper同样采取自监督学习的方式进行跨模态学习，主要解决两个问题：第一，建立了一个用于跨模态检索的网络，将输入的视觉和音频嵌入同一个公共空间内；第二，实现图像的声源部位检测。  

作者的贡献具体如下：1.证明了音频和视觉跨模态嵌入可以被学习，从而解决单模态和跨模态检索任务；2.探索了AVC任务的各种体系结构，比如单图片、多图片或者单图片+多帧光流的视觉流；3.证明了只通过音频特征对图像内的发声物体进行定位是可行的；4.通过实例介绍了一种避免网络快捷学习的数据准备方式。  

AVC和AVTS的区别在nips那篇文章中说的很清楚了，AVTS要求严格的对齐，因此负样本可能会与AVC有所不同，而AVC任务主要判定是视频和音频是属于一个视频中的对齐部分，还是属于两个不同的视频，这样难度看起来会降低一些，负样本也只需要采样不同文件中的样本即可。作者认为，由于网络无法从一帧里捕捉到多帧才会出现的动作信息，因此网络只能通过判断和理解每一个模态中的语义来进行判断。作者在这篇paper中提出了两个网络，第一个网络直接生成适合跨模态检索的embeddings，第二个网络伴随着一个学习过程，用于音源的定位，所有的网络都使用自监督学习方式进行学习。作者使用AudioSet数据集中含有音乐声音的部分，共计110个类，并进行了一系列低质量数据集的预处理。  

## Cross-modal retrieval  

在这一部分，作者将会介绍第一种网络，该网络直接生成适合跨模态检索的声音和视频的embeddings，这里和L³-Net不同的是两个feature不是直接concat，而是让两个embeddings是完全对齐，这样可以进行跨模态查询。那么训练的数据使用hard negative会更好一点。该模型的名字叫做AVE-Net，模型由两个ConvNet子网络分别处理两种模态，然后计算两个embeddings之间的欧氏距离进行判断，之后导入FC层再用softmax输出是否相关联的判别结果，值得注意的是这里使用的是二维卷积核，因为这里并没有把视频的连续帧用于第三维深度表示，这是因为作者并不想让网络学习和识别动作，因为这可能会对AVC的判定有所帮助，而昨天那篇paper中使用了三维卷积，是因为动作识别本身就是其目标下游任务之一。  

该损失函数直接计算两个embeddings之间的欧氏距离，然后根据softmax的输出来判断是否图像和视频是否相关，这个损失函数看起来很类似对比损失函数，但是作者指出两者还是有区别的，具体是:1.不需要调整阈值margin这一超参数；2.显式的输出是否相关联，可以直接和L³-Net相比较，而不是像对比损失函数一样需要额外的超参数确定距离阈值。  

最终的实验结果显示，AVE-Net获得了81.9%的准确率，超过了L³-Net的80.8%，不过学到好的embeddings并不是最终目标，最终作者还是想检测一下跨模态的检索结果。作者使用AudioSet-Instruments测试数据集来评估单模态和跨模态检索表现，使用标准度量 - 标准化折扣累积收益（nDCG）进行评估。它用归一化到\[0,1]范围内的前k个检索项目（k始终等于30，即nDGG@30）的排序列表的质量进行度量，其中1表示完美的排名。不过优于标签是嘈杂的，并且作者只提取每个视频的单帧和1s音频，因此可能错过相关事件，因此理想的nDCG为1是不太可能实现的，只是作为一种最理想的标杆而已。  

作者使用同样用自监督学习方式训练的L³-Net，使用同样的训练方式和训练数据，并同时使用一个与CCA对齐的L³-Net作为baseline，作者还用了一个在ImageNet上预训练的VGGNet作为监督学习的结果对比，结果显示作者的AVE-NET表现在跨模态和单一模态均优于其他所有模型，包括监督学习模型。值得注意的是作者从未在单一模态上对AVE-NET进行训练，但是它却能够很好的完成单一模态上的分类任务，这是由于在映射空间中，同一语义的音频和图像的距离是接近的，这就使得图像特征可以转换成相邻的音频特征，再转换到相邻的其他图像特征，从而完成相似图像的识别。而AVE-NET表现优于L³-Net，是因为该模型是对欧氏距离敏感的，这个模型从设计到训练都是为了检索功能。  

此外作者还探究了多帧对于AVC的影响，对多帧AVE(AVE+MF)和光流AVE(AVE+OF)进行了测试，在AVC任务上得到的结果分别是84.7%和84.9%，有略微的提高，但是在检索任务上并没有性能的提升，作者对此的解释是多帧和光流能够使得网络利用物体的运动信息来进行低级别判断，从而有助于解决AVC问题，但是由于其本身对于两个模态的语义理解并无明显帮助，此外由于促使网络学习动作变换等低层级信息，使得网络相对而言更不倾向于学习高层级的信息。  

作者在这一节的最后提出了一个我觉得非常有意思的观点：需要防止网络进行shortcut，shortcut指网络“偷懒”，学习一些我们不想看到但是又对给定的数据集很有效的信息。举例而言，作者本来对于负样本的处理是随机选择两个不匹配的片段，与相对应的正样本一起导入网络中进行训练，由于片段都是1s，正样本通常随机选择一帧，然后选择以其为中点的1s音频两者组合而成，而负样本则是随机选择1s的音频和一个随机帧组合而成。这种选择方式会导致正样本音频的中点位置通常是与一个帧对齐的，而帧是25fps，这就导致整个正样本的音频时长一定是0.04s的倍数，但是负样本的1s是随机选择的1s，就没有这么严格的对应性。这种正负样本的统计学差异会导致网络更倾向于将音乐时长是0.04的倍数的样本看做正样本，结果虽然正确率能达到87.6%，但是检索的正确率并不高。所以作者对负样本的音频时长也处理为0.04s的倍数，虽然将正确率降低为81.9%，但是在检索的成功率上有所上升。  

## Localizing objects that sound  

接下来就是它对标的下游任务声源定位，即找出一个帧内发出声音的部分。作者将这个问题看做是Multiple Instance Learning的一部分，具体来讲就是将图像分为多个区域，将每个区域提取的embeddings和音频的embeddings对比相似度。在训练的时候仍然和AVC的过程是一样的，只不过在相关联的视听对中，作者添加一个新的方法，从而鼓励某一区域高度相应，从而定位该区域内的对象，这种用一个filter来寻找相关像素块的方式比较类似注意力机制。  

作者将该系统称为Audio-Visual Object Localization Network(AVOL-Net)，整体的结果和AVE-Net还是有不同的，视频子网由AVE-Net的视频子网加上两个1×1卷积层组成，并且没有对conv4_2进行池化，取消了fc层，音频子网和AVE-Net的音频子网相同，此时音频输出是128维向量，视频输出是14×14×128的tensor，计算两者的标量积，得到14×14×1的score map，接着用一个1×1卷积校准，导入sigmoid，对于每个空间位置生成AVC分数输出，输出规模为14×14×1，这个时候就可以根据输出的大小来判断哪一个区域最可能是音源了。接着，使用14×14的最大池化将结果单一化输出，作为AVC的逻辑判断结果。  

最终的结果表明，首先AVC准确率是一样的，表示切换到MIL并不会对准确率和语义概念理解产生损失，其次该网络能够检测广泛的不同视角和尺寸下的发声物体。作者对失败案例进行了分析，发现在无监督学习方法下，网络并不一定会对整个物体进行检测，但是会着重检测那些特定的有辨别力的部分，比如手和钢琴键的接触部分，这其实是一个哲学问题，究竟是人，还是人的手指，还是钢琴本身发出的声音？那么系统应该处理留声机和收音机，因为它们可以发出任意的声音？  

这种问题可以转换为另一个问题，网络是不是只是检测图像中的显著物体？如果这是真的，那么这其实并不是我们期望的结果，只是因为网络中的发声物体一般来说都是显著的，使得网络错误的学习了识别突出的物体而非发声物体。作者为了测试这一猜想，将不对应的视频帧和图像合在一起询问网络声源是哪个物体，如果声音即使不关联，帧中突出的物体仍然被识别，那么说明猜想正确，模型训练有误。但是结果发现猜想是错误的，当在一个小提琴图像上播放敲鼓的声音时，网络显示图像上无物体发声，而当另外一个小提琴发声的时候，网络显示图像中的小提琴发声，这说明网络的确可以学习到一个图像中不同物体的发声方式并进行识别的。  

作者将一个只选择网络正中央物体的对照模型作为baseline……结果显示baseline的准确率是57.2%，AVOL-Net的准确率是81.7%。  

## Conclusion  

作者设计了一个有效的自监督学习AVC模型，并在两个下游任务视听检索和声源定位中得到了很好的表现，视听检索的表现甚至超过了监督学习方法。作者接下来会考虑对声源定位模型添加显示的软注意力机制，用来替换目前简单的最大池化层，以希望提高最终的表现。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

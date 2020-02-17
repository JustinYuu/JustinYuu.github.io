---
layout: post
title: "PyTorch-CIFAR10-Networks"
description: "Notes"
categories: [Project]
tags: [Python]
redirect_from:
  - /2020/02/17/
---

# PyTorch-CIFAR10-Networks  

## Introduction  

由于疫情在家里实在没啥事做，又不想天天看论文，所以就决定手撸一遍现在的主流分类网络，就当了解网络的细节了，顺便提升下代码能力。全程一共花了两个星期多一点，实现了17个网络架构的45个网络。  

## Implementation Details  

由于实验室只分给我两个卡，所以我不可能去跑ImageNet，就选了相对来说小很多的CIFAR-10数据集，跑起来也挺费劲，但是总之还可以接受。  

网络方面选了我能跑动的几乎所有流行的分类网络，至于GPipe和NASNet这种东西我就不碰了，PNASNet这种搜出来的东西我也不想去试，没啥大意思。  

实验室gpu型号为1080Ti，python版本为3.6.10，cuda版本9.0，PyTorch版本1.1.0，torchvision版本为0.3.0。  

训练的方法和参数基本上完全相同，优化器使用SGD，epoch为300（少数几个网络是200，不过这些200的网络在接近200epoch的时候loss已经为0.001了，所以准确率也差不到哪里去），momentum=0.9, weight_decay=0.0001，学习率为0.1，每过100个epoch除以10，总体来说还算中规中矩。  

之所以使用相同的训练方法，首先是因为要对比网络的准确率，所以要控制一下变量，其次是因为有些网络的训练方法也太复杂了，有的可以用奇淫技巧来形容，比如EfficientNet，我把它的训练方式摘抄一部分：initial learning rate 0.256 that decays by 0.97 every 2.4 epochs. 我实在不明白他们是怎么调出来这种奇葩参数的，0.256的学习率用网格搜也得搜到千分位，属实牛批。  

不过使用同样的训练方法会导致有些网络的性能无法完全的发挥出来，所以最后的比较结果也不是网络的最优表现，只是在这一比较常规的训练方式下的表现。  

## Results  

由于网络是在太多了，我列了个表在下面。  

model | accuracy(×100%) | epoch 
:-: | :-: | :-: 
ResNet18 | 0.9461 | 200
ResNet34 | 0.9507 | 200
ResNet50 | 0.9418 | 200
ResNet101 | 0.9471 | 200
ResNet152 | 0.95 | 200
VGG11 | 0.9157 | 200
VGG13 | 0.9032 | 200
VGG16 | 0.9359 | 300
VGG19 | 0.9347 | 300
AlexNet | 0.8195 | 300
LeNet | 0.7155 | 300
GoogLeNet | 0.9477 | 300
MobileNetV1 | 0.9185 | 300
ShuffleNetV1_g1 | 0.9145 | 300
ShuffleNetV1_g2 | 0.9123 | 300
ShuffleNetV1_g3 | 0.9205 | 300
ShuffleNetV1_g4 | 0.9175 | 300
ShuffleNetV1_g8 | 0.916 | 300
MobileNetV2 | 0.9434 | 300
ShuffleNetV2_Z05 | 0.9009 | 300
ShuffleNetV2_Z1 | 0.926 | 300
ShuffleNetV2_Z15 | 0.9338 | 300
ShuffleNetV2_Z2 | 0.9382 | 300
DenseNet121 | 0.9476 | 300
DenseNet169 | 0.9486 | 300
DenseNet201 | 0.9476 | 300
DenseNet264 | 0.9502 | 300
PreActResNet18 | 0.94 | 300
PreActResNet34 | 0.9474 | 300
PreActResNet50 | 0.9525 | 300
WRN_16_4 | 0.8189 | 300
WRN_40_8 | 0.9119 | 300
WRN_28_10 | 0.912 | 300
ResNeXt50_8x14d | 0.9536 | 300
ResNeXt50_1x64d | 0.9447 | 300
ResNeXt50_32x4d, |0.9548 | 300
ResNeXt50_2x40d | 0.9505 | 300
ResNeXt50_4x24d | 0.9533 | 300
SEResNet | 0.9437 | 300
SqueezeNet | 0.9297 | 300
EfficientB0 | 0.948 | 300
DPN92 | 0.9521 | 300

分析一下，准确率最高的是ResNeXt50_8x14d，准确率能达到95.36%，当然之前提到过了，这只是网络在这一特定训练方式下的表现，不代表网络的最高水平。  

总体来看MobileNet, ShuffleNet, SqueezeNet这种轻量化网络能够达到90%以上的准确率，训练还挺快，还是非常靠谱的，ResNet和他的兄弟姐妹们的表现还是非常的稳定且优秀，DenseNet这种大型网络的训练结果也还不错，但是WRN的表现倒是挺一般，不知道是不是我实现的有问题。  

此外PreActResNet101和PreActResNet152两个网络在训练的过程中loss一直不下降，但是使用同样的瓶颈层的PreActResNet50却能正常训练，不知道是我实现的有问题还是数据集不合适，反正找了好久没找到问题在哪里。  

新开了一个repo，把代码放在里面了，这里是[传送门](https://github.com/JustinYuu/pytorch-CIFAR10-playground)。  

## Conclusion  

手撸一遍这些网络对于网络的理解是绝对有帮助的，至少知道了不同网络的区别和创新点在哪里，其实看起来novelty也就这样，有些变种修改的也不是很多，但是作者能在数学和理论上进行一大堆证明，不知道是真的理论方面改进之后改的网络，还是改网络乱碰之后强行添加上的理论证明，总之性能的确能提升，这确实挺厉害的。  

不过对于编程能力的提升还是有限的，因为很多网络的区别也就是实现细节的不同，比如卷积的参数不同啦，BN和激活函数的位置不同啦，各种各样的残差等等，充其量也就是对于分类网络的搭建熟练了一些，但是对于其他方面的编程，比如目标检测语义分割，包括强化学习GAN等等，压根没有任何帮助。  

总之分类网络就先告一段落了，接下来研究研究小样本的分类，顺便复现几个目标检测的经典算法看看。在家里闲着实在太难受了，希望疫情早日过去，武汉加油。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

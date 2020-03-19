---
layout: post
title: "Daily Paper 50: MAML"
description: "Notes"
categories: [CV-Meta]
tags: [Paper]
redirect_from:
  - /2020/03/19/
---

# Daily Paper 50: Model-Agnostic Meta-Learning for Fast Adaptation of Deep Networks  

## Introduction  

这一篇来头可大了，是Model-Agnostic领域的开山之作，第一作者是元学习领域的权威Chelsea Finn，发表在ICML2017上。这篇paper的主要思想是通过一系列训练任务的学习，可以调整到最为出色的模型参数，从而使得模型能够在少量的梯度更新后就能达到很好的表现。作者认为这一算法能够在小样本图像分类和强化学习上有很好的表现。  

这篇paper承接前面的Optimization as a model，也是注重学习优化本身，而这里的侧重点又偏向于任务的参数初始化。这里并不像之前的Optimization as a model一样，需要扩充大量的学习参数，也不需要对模型架构有任何的约束，因此可以和多种网络和损失函数相结合，提升学习的效果。  

## Method  

元学习的目标本来就是为了训练一个可以泛化到一些新任务上的模型，所以作者首先对任务进行了定义。作者将模型记为f，输入为x，输出为a，在meta-learning阶段，模型通过一大批任务中进行学习，每一个任务都有一个损失函数L，一个基于初始观察的数据分布q(x<sub>1</sub>)，一个过渡分布q(x<sub>t+1</sub>\|x<sub>t</sub>, a<sub>t</sub>)和一个episode的长度H。  

在作者的元学习情景中，他们想建立一个在任务之上的分布p(T)，使模型去学习这一分布。训练集中的任务和测试时的任务都将从p(T)中取出。  

作者的目光着重聚焦在θ上面，他们认为模型中的一些参数θ是对任务的改变敏感的，即这些参数稍微改变一些之后会改变梯度更新的方向，从而导致损失函数的值会有极大的变化。作者的模型也就是要改变这些影响重大的参数，从而使得少量的梯度更新步骤就可以对任务的学习产生巨大的影响。这一步作者将其称之为meta-optimization，不同task的源优化过程使用SGD进行，整体的流程见下图:  

![50-1](/images/daily paper/50-1.png)  

MAML的方法使用了二次梯度，这就需要二次求导，也就是使用Hessian矩阵，这可以用主流的深度学习框架实现。  

## Species of MAML  

在不同类型的任务中，MAML的细节有所不同。在监督学习的回归和分类中，模型只接受单一的输入，也输出单一的输出，因此H=1，损失函数回归一般使用MSE，离散分类任务使用交叉熵。  

强化学习中，小样本元学习的认识是使得智能体能够快速的从一个新任务中通过少量的测试经验就可以学会新的policy。整个RL任务可以看做是一个马尔科夫决策过程，模型学习到的f<sub>θ</sub>是一个从状态x映射到每个时间步t的动作a<sub>t</sub>。在K-shot强化学习中，从任务列表T中选出K个认为，每个任务的reward可用作新任务T<sub>i</sub>的使用。由于强化学习的reward是不可导的，所以使用policy gradient进行处理模型梯度和meta-optimization。由于policy gradient是on-policy算法，在f<sub>θ</sub>学习过程中的每一个额外的梯度步都需要从当前的policy f<sub>θ</sub>中选择新样本。小样本监督学习和强化学习的算法如下图所示：  

![50-2](/images/daily paper/50-2.png)  

## Experiment  

作者的实验主要想证明三点：MAML能不能快速的学习到新任务，以及MAML能不能在多种领域的元学习中都派上用场，和用MAML进行训练的模型会不会随着梯度的不断更新而持续性的提高表现？作者随即做了大量的实验。  

对于回归问题，作者发现使用MAML的MSE显著低于不使用的，说明MAML有助于减少小样本回归的过拟合。  

对于分类问题，作者在Omniglot和miniImageNet上进行了训练，结果证明在各个领域的各个任务上都实现了STOA，超越了当前所有的metric learning方法和memory-based方法。  

对于强化学习问题，作者在2D Navigation和Locomotion上进行了实验，结果显示MAML可以更快速的学习新的目标速度和方向，在两到三个梯度更新后就能够有很好的表现。  

## Conclusion  

这篇paper可谓有划时代的意义，它证明了对参数本身的优化，以及对其他任务的学习，可以对新的任务产生巨大的影响，从而在少量样本和少量梯度更新的情况下，能够达到很好的效果。相对于在这之前的度量学习而言，这种学习方式才是真正的learning to learn，其二次优化的思想也是开创性的。总之MAML开创了元学习领域的新方向，论文中的实现细节也很详细，可以尝试复现一下分类的那部分。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

---
layout: post
title: "Daily Paper 46: Optimization as a model for few-shot learning"
description: "Notes"
categories: [CV-Meta]
tags: [Paper]
redirect_from:
  - /2020/03/06/
---

# Daily Paper 46: Optimization as a model for few-shot learning  

## Introduction  

这篇paper是twitter的两个学者发表在ICLR2017上的。我认为严格来讲不算是metric learning方法，反而更像是model based方法。作者用了一个基于LSTM的meta-learner模型去学习特定的优化方式，进而训练另一个用于小样本学习的learner网络。我觉得读到这篇paper，才真的有点元学习的意思了，之前的metric learning感觉都不太接近元学习的方式，这里的两步法更像是当今meta learning的一般步骤的雏形。作者认为他们搞出来的这个meta-learner可以通过特定的子任务学习到短期知识，同时又可以通过所有任务学习到一些共性的长期知识。这里学习的其实就是参数的更新和模型的初始化，也就是说元学习学到的方法就是如何训练子任务网络，读到这里的时候我其实不是很明白为什么参数的初始化和更新由机器设置就能够解决小样本学习的问题了，还是要看作者的进一步阐述。  

## Task Description  

这里的元学习定义和训练方式与之前并无太大差异，对于K-way N-shot而言，就是训练集有K个类，每个类有N个图片，测试集每一类都有一张图片，测试集和训练集组成一个子任务，也叫做一个episode。  

将视角再放大到整个元学习模块中，其实又可以分为三部分，即元学习的训练、验证和测试，元学习训练部分主要用来训练一个学习过程，使得学习过程可以使用episode中的训练集进行训练，在测试集上取得好的效果，也就是使用episode进行训练，学习一个通用的学习过程。元学习验证部分主要用来选择超参数，测试部分用来评价泛化性能，这就和一般的深度学习方式差不多了。也就是说，整个任务最大的区别就是学习的不是特定的分类，而是一种通用的分类方式，表现形式为参数的更新和初始化。  

## Model  

整体的流程可以用一个算法流程图来表示:  

![1-1](/images/daily paper/Optimization-as-a-model.png)  

整体的流程还是相当清晰的，一共进行n个episode，在每一个episode中，都进行T次循环，来进行learner的学习，更新learner的参数，在每一个episode进行完之后，进行meta-learner的更新，总之就是两层更新。  

对于learner的更新，使用的是标准梯度下降法的变体，即θ<sub>t</sub> = θ<sub>t-1</sub> - α<sub>t</sub>▽θ<sub>t-1</sub>L<sub>t</sub>，其中θ是learner的参数，▽θ<sub>t-1</sub>L<sub>t</sub>是损失相对于参数θ<sub>t-1</sub>的梯度，这种更新方式灵感来源于LSTM中的单元状态更新，LSTM的单元状态公式为c<sub>t</sub> = f<sub>t</sub>⊙c<sub>t-1</sub> + i<sub>t</sub>⊙c<sub>t</sub><sup>~</sup>, 在这里作者使用一个元学习器LSTM来作为meta-learner，将LSTM的细胞状态f<sub>t</sub>设置为学习器的参数θ<sub>t</sub>，将候选细胞状态c<sub>t</sub><sup>~</sup>设置为▽θ<sub>t-1</sub>L<sub>t</sub>，定义i<sub>t</sub> = σ(W<sub>I</sub> · \[▽<sub>θ-1</sub>L<sub>t</sub>, L<sub>t</sub>, θ<sub>t-1</sub>, i<sub>t-1</sub>] + b<sub>I</sub>)，作者认为这样可以使得meta-learner很好的控制速度，能够在避免分歧的同时快速的训练分类器。对于f<sub>t</sub>，作者貌似并不想像LSTM的一般情况一样取1，而是定义为f<sub>t</sub> = σ(W<sub>F</sub> · \[▽<sub>θ<sub>t-1</sub></sub>L<sub>t</sub>, L<sub>t</sub>, θ<sub>t-1</sub>, f<sub>t-1</sub>] + b<sub>F</sub>)。作者认为c<sub>0</sub>，即LSTM的单元状态的初始值，应该作为meta-learner学习的参数之一，这也就是上面说的参数初始化方式，总体来说还是学习了参数的初始化和更新方式。  

此外，作者使用了一个较为紧凑的LSTM模型，在学习器梯度的坐标上共享参数，换言之，每个坐标都有自己的隐藏值和单元状态值，但是所有坐标的LSTM参数都是相同的。这样做可以使得训练过程中只需训练一套LSTM参数，但是每一个坐标又可以依赖于自己坐标的历史来进行优化。作者还采用了andrychowicz等人提出的预处理方法，在梯度的维度和每个时间步的损失时应用该方法，具体方法是当\|x\|≥e<sup>-p</sup>的时候，x→(log(\|x\|)/p, sgn(x)), 其他时候x→(-1, e<sup>p</sup>x)。  

此外还有一些训练的细节，包括BN的使用，LSTM学习器的初始化方式，以及采用的梯度独立假设从而避免使用二阶导。  

## Experiment  

对于learner，作者使用了4层的CNN，包括BN, ReLU和最大池化，而对于meta-learner，作者使用了2层的LSTM，第一层是普通的LSTM，第二层是按上述方法修改过的LSTM，优化器使用Adam，学习率设置为0.001，并使用gradient clipping，值为0.25。  

实验是在miniImageNet上跑的，作者选了matching network作为baseline，由于当时2017年的小样本baseline是在太少，所以作者由自创了两个baseline，第一个叫做Baseline-nearest-neighbor, 用作者的训练网络生成训练集的embedding后，使用最邻近算法来进行测试集的分类；第二个叫做Baseline-funetune，这是一个粗糙版本的meta-learner。  

最后的结果表明，作者在5-class 5-shot的表现上显著优于其他baseline，在1-shot的表现上与matching network(FCE)类似，也达到了STOA的水平。此外作者还对网络进行了可视化，结果就不再赘述了。  

## Conclusion  

总之，作者搞了一个基于LSTM的元学习模型，该模型相对于其他度量学习方法而言，真正做到了学习学习方法本身，还是值得称道的。此外，源代码其实并不复杂，github上的一些复现代码读起来还是挺容易理解的，自己复现起来应该也不会太困难。  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

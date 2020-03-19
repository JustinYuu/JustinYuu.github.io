---
layout: post
title: "Stanford CS330 Lecture 2 - Multi-Task & Meta-Learning Basics"
description: "Notes"
categories: [CV-Meta]
tags: [Python]
redirect_from:
  - /2020/03/18/
---

# Stanford CS330 Lecture 2 - Multi-Task & Meta-Learning Basics  

今天学习第二节课的内容，从plan for today看起来内容还是挺多的。  

## Multi-Task Learning  

### Some Notation  

首先介绍一些标注的术语，方便后续教学的进行。首先是基本的模型，f<sub>θ</sub>(y\|x)，以及单任务学习(监督学习)的一般表示:D={(x,y)<sub>k</sub>}, min<sub>θ</sub>L(θ, D)。一般的采用的损失是负对数似然negative log likelihood，公式为L(θ,D)=-E<sub>(x,y)~D</sub>\[logf<sub>θ</sub>(y\|x)]。对于task的定义，上节课已经说过一些了，这里给了一个更为精确的定义：J<sub>i</sub> := {p<sub>i</sub>(x), p<sub>i</sub>(y\|x), L<sub>i</sub>}，所对应的数据集为D<sub>i</sub><sup>tr</sup>, D<sub>i</sub><sup>test</sup>，一般会将tr省略，直接用D<sub>i</sub>来定义训练集。  

对于多任务分类，L<sub>i</sub>在所有的任务中都是相同的，比如手写语言识别等，对于multi-label learning来说，L<sub>i</sub>，p<sub>i</sub>(x)对于所有的任务来说是相同的，比如CelebA attribute recognition，或者场景理解等。那么问题就来了，L<sub>i</sub>啥时候不相同呢？答案就是当任务中的标签既有离散的，又有连续的，或者在这些任务中你更关注某一个。  

此外，有的时候会添加一个叫做task descriptor，那么这个时候函数就不再是只关于数据集x的函数了，task descriptor z<sub>i</sub>也作为一个自变量存在，即f<sub>θ</sub>(y\|x,z<sub>i</sub>)。z<sub>i</sub>可能含有task索引的one-hot向量，也有可能含有一些meta-data，比如一些进一步的描述。那么最终的目标就是min<sub>θ</sub>∑<sub>i=1</sub><sup>T</sup>L<sub>i</sub><sub>i</sub>(θ, D<sub>i</sub>)。  

### Models  

那么接下来的任务就比较清晰了，如何建立模型，包括如何考虑条件z<sub>i</sub>，以及如何优化训练过程，即model和training的选择。这里举了一个例子，如果z<sub>i</sub>就是任务的索引，那么如何建模从而使得不同的任务之间共同的信息越来越少？这里给出的一个可能的方案是根据index来用不同的网络进行训练，这样可以最大程度的避免共享参数。这是一个比较极端的例子，在另一个方面比较极端的例子是完全使用同一个网络，对于z<sub>i</sub>只是简单将z<sub>i</sub>和输入值或者激活值链接到一起，这样所有的任务的参数都是完全共享的，除了z<sub>i</sub>后面的FC层可能会根据z<sub>i</sub>的不同而有稍许不同，当然也可以将每一层都加上z<sub>i</sub>的索引，这样task
-specific的参数将会更多。  

从上面的例子中我们可以发现参数θ其实可以分为两部分，即共享参数θ<sup>sh</sup>和task-specific参数θ<sup>i</sup>，在优化的时候将参数θ分开优化，对于共享的部分，只需要优化一次，而对于不同的部分，再根据任务不同来分别优化，这样看起来似乎会更省时间。那么再将视角回到z<sub>i</sub>本身，我们发现其实任务的不同就是根据z<sub>i</sub>的不同带来的，所以我们决定如何处理z<sub>i</sub>，其实就是决定哪一部分是共享参数，以及如何共享这些参数，与z<sub>i</sub>无关的即为确定的共享参数。  

那么进而介绍了一些可能的方法，比如基于concatenation的方法，基于additive的方法，也就是对于z<sub>i</sub>到底该怎么办。仔细想想这两个其实差不多，就是位置的不同而已。此外还有multi-head架构，即经过一些共享层之后，再根据层的不同分开。除了相加，还有相乘的方法，将conditioning representation缩放到相同尺寸后再进行相乘操作。这种方法是非常昂贵的，大家一向认为加法是比乘法要快的，比如今年华为的加法神经网络，但是也是有好处的，比如可以轻松的实现gating的功能，从而可以像multi-head一样指定哪一部分特定的处理哪些任务，这样对于multi-head中不同的head，以及独立的网络而言泛化性能良好。这门课相对于CS231n不太好的一点是学生的提问很多不会在字幕里出现，听又听不清，导致我只能从结果推断问题是什么，感觉错过了很多好问题。这里学生的提问貌似是这和注意力机制是不是有关联，我觉得这是一个很好的问题，Finn觉得可能看起来很像，但是操作方式不一样，一个是dot product，一个是element-wise multiply，但是两者都有差不多的意思吧。  

当然z<sub>i</sub>作为一个one-hot向量只是一个例子，如果想完全泛化一个新任务，那么可能一个one-hot向量是不够的，毕竟需要一些与这些任务相关联的信息。更加复杂的例子有很多，比如说Cross-Stitch Networks, Multi-Task Attention Network, Deep Relation Networks, Sluice Networks等等，这些在后面会提到。  

### Optimizing  

接下来就是优化的方法，也就是训练方法。最为基础的方法如下：首先对任务们进行小批量采样，然后对每一个任务进行小批量的数据点采样，然后计算小批量的loss，接下来BP来计算梯度，最后用一些经典好用的优化器来优化梯度。这种方法的好处是可以无视不同任务数据集的数量，能够用一种统一的标准方式进行采样，这种保证每一个任务中的数据规模相同的要求是很重要的，尤其是对于回归问题而言。  

方法就介绍到这里，我估计其他进阶内容应该要之后再讲了。  

### Challenge  

此外当今还有一些挑战。首要的挑战叫做negative transfer，有些时候独立的网络可能会表现的更好，这种现象的原因是有些任务的数据可能会损害另一些任务的表现。这里举了一个Multi-task CIFAR100上的例子，作者测试了一下当前multi-task的STOA方法，结果发现独立的网络比这些方法用起来结果更好。这种原因是多方面的，可能是由于优化的问题，比如不同任务的梯度之间的影响，或者不同任务之间的学习速度不同；也可能是由于表示容量的限制，multi-task由于要学习一些更多的表示，所以网络一般比单任务网络需要大一些，否则容易欠拟合。这里有学生提出了我也产生了的疑惑，那就是你一个网络来解决十个任务，比十个网络解决十个任务表现差，难道不是正常的吗，这难道不是因为更省事更方便才使用multi-task的吗？Finn的回答是可能十个网络解决十个任务，那么数据集规模就不够了，所以才需要用multi-task来从别的任务的数据集中学习一些可能与自己的任务有关的信息。所以multi-task严格上来讲效果应该比single-task要好的，因为数据集扩充了若干倍。那么学生继续问了，如果数据集完全相同，只是标签可能根据任务的不同有所不同的情况下，是不是multi-task就没啥作用了？毕竟之前看出的作用就只局限在数据共享上，这里Finn的看法是multi-task还是有效的，因为共享数据并不是简单的共享输入，而是共享监督方法，因为不同任务的监督方法可能不同，即使同样的数据学到的信息还是会有所不同。此外还有学生提问，大网络究竟比小网络好在哪里？这里除了优化的时候可能会压力更小之外，大的网络可以同时学到更多的任务表示，因为multi-task的多个task都需要学习自己的表示，并且这是并行的而不是串行的，所以就需要更为强大的表示学习能力，这体现在网络的规模大小上。回到negative transfer上来，Finn认为遇到这种问题最好是在任务之间共享更少的参数，毕竟这并不是一个二元的决定，你可以自己决定共享多少信息。  

还有一个挑战叫做Overfitting，过拟合可能发生在某些任务中，这是由于信息可能分享的不够多所致，分享更多的信息会起到过拟合的作用。  

### Case Study of real-world multi-task learning  

Finn进而举了一个例子，是YouTube的推荐算法，其算法的目标是为YouTube提供用户视频的推荐。那么可选的目标有很多种，比如用户打分很高的视频，用户分享的视频，或者用户喜欢看的视频。此外还有一些隐藏的因素，比如用户是否已经看过了，这被Finn称作feedback loop。那么对于这个案例，框架的输入就是用户的特征和当前正在看的video，然后要通过模型生成一系列候选视频，然后再对视频排序，最后将最顶部的视频推荐给用户。那么显然重点在于候选视频的选择以及ranking。这里作者的想法是把大量的有关联的视频作为候选视频，然后将模型的处理重点放在ranking上面，ranking需要考虑engagement和satisfaction本身，engagement参与度包括用户是否点击过这些视频，花了多少时间在这些视频上，前者是一个二元分类问题，后者是一个回归问题。satisfaction即为满意度，比如用户是否点击了like按钮，或者用户的评分，前者是一个二元分类问题，后者是一个评分。那么将engagement和satisfaction合在一起综合考虑，就是最后的ranking score，这里可以使用手动设置的加权相加。  

Finn认为该例子网络的架构可以使用Shared-Bottom Model，即一个Multi-head架构，不过这种方法可能在任务之间的相关程度很低的时候有负面的影响。因此在实际情况中，一般都会使用一些替代信息，比如MMoE，即Multi-gate Mixture-of-Experts，该模型允许网络的不同部分可以specialize专家神经网络f<sub>i</sub>x，使用一个函数g(x)来决定哪一个input是要使用的，然后再使用选择的专家来计算特征。  

在实验部分，训练一般按照时序进行，在TPU和Tensorflow上进行训练，使用AUC和均方根误差来进行性能评价。  

## Meta-Learning  

接下来进行元学习的部分，这里主要学习三方面，problem formulation, general recipe of meta-learning algorithms, Black-box adaptation approaches。  

### Problem Formulation  

有两种观点看待meta-learning，一种是mechanistic view，一种是probabilistic view。mechanistic观点认为深度神经网络模型可以读取整个数据集，并在新的数据点上进行预测。训练这种网络需要使用元数据集，其中包含了多种任务的子数据集。这种观点能够使得元学习算法的实现更简单。probabilistic观点认为从一系列任务中提取先验信息可以使得新任务的学习更有效。使用先验的训练集去推断最为可能的后续参数的形式，从而学习一个新的任务，这种观点会使得元学习算法更容易被理解。  

对于元学习的数据集，一般认为会有两部分，D={(x<sub>1</sub>, y<sub>1</sub>),...,(x<sub>k</sub>,y<sub>k</sub>)}, D<sub>meta-train</sub> = {D<sub>1</sub>,...,D<sub>n</sub>}，那么元学习的过程可以写为argmaxlogp(φ\|D,D<sub>meta-train</sub>)。但是这也会有一些问题，比如我们不想永远把元训练数据集放在身边，这样会导致数据集规模太过庞大，这时候我们可以学习一个meta-parameters θ:p(θ\|D<sub>meta-train</sub>), 这一参数就表示需要从元数据集中学习到的用于解决新问题的知识，那么之前的logp(φ\|D,D<sub>meta-train</sub>)就可以写成log∫<sub>θ</sub>p(φ\|D, θ)p(θ\|D<sub>meta-train</sub>)dθ ≈ logp(φ\|D, θ<sup>\*</sup>) + logp(θ<sup>\*</sup>\|D<sub>meta-train</sub>), 其中θ<sup>\*</sup> = arg maxlogp(θ\|D<sub>meta-train</sub>)，对于θ<sup>\*</sup>的求解，是我们真正要处理的元学习任务。  

 


---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

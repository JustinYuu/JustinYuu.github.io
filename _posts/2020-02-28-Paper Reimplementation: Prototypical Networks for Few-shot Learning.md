---
layout: post
title: "Paper Reimplementation: Prototypical Networks for Few-shot Learning"
description: "A pytorch reimplementation of Prototypical Networks for Few-shot Learning"
categories: [CV-Meta]
tags: [Reimplementation]
redirect_from:
  - /2020/02/28/
---

# Paper Reimplementation: Prototypical Networks for Few-shot Learning  

## Introduction  

这次复现的小样本方向的文章也是非常经典的一篇，叫做prototypical networks，是由多伦多大学和twitter发表在NeurIPS2017上的，主要修改了一下matching network，将一类图片抽象成一个prototype，然后用新的图片去对应这些prototype，整体的思想类似于最邻近算法，但是还是有一些细节上的trick的，文章思路清晰，简洁明了，我个人认为是一篇不可多得的好文章。  

Matching network是一个经典的小样本分类网络，下一篇就会复现这篇网络。这里其实只是对matching network做了一个简单的修改，将计算支撑集和查询集之间的距离变成了计算支撑集生成的prototype与查询集之间的距离，从余弦相似度改成了欧氏距离，效果得到了很好的提升。这看起来是简单的修改，但是其实是有数学原理在内的，下面展开叙述一下。  

## Model  

在数学领域中，有一个叫做regular Bregman divergences的东西，Bregman散度是一类度量方法的集合，它类似一种距离，用来度量两组向量之间的差异性，具体来讲，它其实是基于一阶泰勒展开而产生的，在坐标系上表示为函数f在x点处的函数值与在y点处一阶taylor近似的差，即div(f)(x,y)=f(x)-\[f(y)+f'(y)(x-y)]。那么Bregman散度满足一个特性，即当若干个点以任意概率分布在某个特征空间的时候，这些点的平均值一定相较于这若干个点的平均距离最小，这里的距离是由Bregman散度定义的任意距离。这里可定义的有很多，因为很多函数都可以用类似的方式来定义，比较常见的有欧氏距离、KL散度、马氏距离等，深入的理论可以看一看最优化相关的知识。  

那么有了这样一个数学知识，作者就进一步想到，这种数学上的严格“平均值”能不能作为整个特征空间组成的集合的represention呢？这其实就是给所谓的“取平均”提供一个严格的数学证明，我认为作者大概率是先取平均发现有效之后才想到用Bregman散度证明的，毕竟取平均来做代替是一个非常容易想到的方法，那么对于每一个support set，都会用神经网络映射到一个高维特征空间中，然后对于所有support set的映射取平均值作为这一类的prototype，把所有的support set全部计算完毕之后，得到所有类别的prototype，然后用相同的神经网络映射query set，然后计算query set每一张图片的映射结果和prototype之间的距离，这里用的是欧氏距离，作者应该是试了一下，发现欧氏距离效果最好。那么作者将距离作为优化的目标，训练多个episode之后，使神经网络能够把图片映射到最合理的位置，从而达到准确的识别。  

作者使用了一个四层的卷积神经网络作为映射网络，每层后面都有BN和2×2的池化层，使用ReLU作为激活函数，在Omniglot, miniImageNet上进行了5-shot和1-shot的测试，都取得了不错的准确率，也在CUB-200 2011上进行了zero-shot的测试，能够达到当时STOA的准确率。  

## Implementation Details  

这个模型看起来非常简单，但是实现起来花了我相当多的时间，因为我之前的水平仅限于重写dataset和按部就班的进行训练，并没有接触过pytorch的很多重写方法，这一次完整的重写一次收获还是很多的，下面简单说一下实现细节。  

复现基于pytorch，首先要重写dataset，如果是Omniglot数据集，需要将每个语种的细分字母各自看做一个类导入数据集中，如果是miniImageNet则按照二级目录遍历就好。之后需要重写dataloader，由于这里每一个episode都需要nc个类的nq个query set和ns个support set，那么在每一个episode，dataloader都需要返回一个规模为nc×(nq+ns)的子数据集，那么这个时候dataloader的默认sampler和batchsampler就无法满足这一需求，需要重写一份sampler。我的写法是在每一个sample的过程中都返回一个规模为nc×nq+ns的索引，索引是一个元组(cls_idx, img_idx)的形式，dataset的__get_item__魔术方法可以根据收到的索引直接返还到对应的图片和其标签。由于__get_item__方法仍然只返回一个值，所以dataloader的batch设置为nc×(nq+ns)，也就是sampler返回索引的规模，也就是一个episode所需的图片的总数。需要提一下，这里的label不需要返回原本的类标签，由于每一次都是一个全新的任务，除了网络参数在不断迭代，其他的support set和query set都是新的，随机选的nc个类别也都是新的，每一个episode的数据之间也没有什么练习，所以只需要将标签从0到nc之间arrange一下就可以了。  

搞定了数据的载入，下面要考虑prototype的生成。由于文章中提到要先将nc个prototype计算完毕，所以要先把nc个类的support set导入神经网络中，将每一个类的ns个embeddings取平均得到该类的prototype，一共生成nc个prototype，这里可以用python的向量化完成，从而节省大量的时间。接下来再将nc个query set导入神经网络中，各自取得其图片对应的embeddings，然后计算距离所有类别的prototype的欧氏距离，使用SGD来优化距离，通过训练使得神经网络能够将图片映射到高维空间中里该类别的prototype更近的位置。这样一共可以得到nc×nq×nc个距离，代表了nc个类别中nq个query set图片距离nc个类别prototype的距离。使用log_softmax优化距离的第三维度，即单个embedding距离每个类别的距离，第i个类别的损失即为softmax的第i个维度的值，也就是-log_softmax(-dist)[i,:,i].mean(1)，最后可以得到nc个值，再取平均即可得到整个子数据集中的loss平均值了，通过不断的学习，不断的降低loss值。  

在测试的时候，根据原论文的意思，训练的nq应该需要和shot的值相同，但是way不需要相同，训练的过程中nc的值可以达到60，但是在测试的时候进行了5-way 1/5-shot和20-way 1/5-shot四种实验，作者给出的理由是训练的时候nc大一点训练更容易。  

## Result  

作者在Omniglot上的结果是96%以上，在miniImageNet上5way的结果是49和68，在CUB-200上的zero-shot结果是54.6%，这个准确率在当时还是相当高的。我的结果还没跑完，跑完了会列个表放上来。  


---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

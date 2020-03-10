---
layout: post
title: "Numerical Optimization Note Chapter 2"
description: "Note of Numerical Optimization"
categories: [Numerical Optimization]
tags: [Math]
redirect_from:
  - /2020/03/10/
---

# Daily Paper 44: Numerical Optimization Exercise Chapter 2  

## Introduction  

本来我以为看吴立德教授的课就可以了，没想到他讲的实在是有点混乱，需要用大量的时间理解和整合，所以还是要自己结合大黄书梳理一遍记个笔记，不然看过之后根本没法回头重新回顾。  

笔记从第三章开始，前两章主要讲了一些整体性的知识，属于提纲挈领的内容，这里就略过不表了。这里由于github pages的插入公式实在是太不友好，我就尽量放图片，一些简单的公式就手打了，应该也看得懂。  

第三章介绍线搜索，内容还是很多的。所谓的线搜索，也叫做一维搜索，是最优化中的一种基础方法，它的主要步骤可以用一个公式来概括，即x<sub>k+1</sub> = x<sub>k</sub> + α<sub>k</sub>p<sub>k</sub>。在这个公式中，α<sub>k</sub>代表步长，p<sub>k</sub>代表方向，每一个迭代过程中x都会向着优化方向移动指定的步长，直到达到局部最优。这里线搜索需要保证局部最小值存在在自己指定的范围之中，也就是左右界必须确定才行。  

线搜索其实是一个广义的概念，大家一般还是称呼线搜索的一些子方法，比如牛顿法、最速下降法等等。这里不同的方法的方向选择不同，对于最速下降法（反梯度法），p<sub>k</sub><sup>G</sup> = -▽f(x<sub>k</sub>)，对于牛顿法，p<sub>k</sub><sup>N</sup> = -▽²f(x<sub>k</sub>)<sup>-1</sup>·▽f(x<sub>k</sub>)，对于拟牛顿法，p<sub>k</sub><sup>B</sup> = -B<sub>k</sub><sup>-1</sup>▽f(x<sub>k</sub>)，此外吴立德教授还提到了共轭梯度法，不过没有说具体形式，只是说后面会讲到，那么这里就不提了。  

## p<sub>k</sub>的选择  

对于p<sub>k</sub>的选择，不同的方法有着不同的选择方式。首先定义ψ(α)=f(x<sub>k</sub>+αp<sub>k</sub>)，此函数是以α为自变量，p<sub>k</sub>和x<sub>k</sub>都是已知的值，那么α的取值应该是ψ的全局最优解。  

对于该函数，很明显有ψ(0)=f(x<sub>k</sub>)，简写为f<sub>k</sub>, ψ'(α) = ▽f(x<sub>k</sub>+αp<sub>k</sub>)<sup>T</sup>p<sub>k</sub>, ψ'(0)=▽f<sub>k</sub><sup>T</sup>p<sub>k</sub>, p<sub>k</sub>是下降方向，所以ψ'(0)需小于0，即▽f<sub>k</sub><sup>T</sup>p<sub>k</sub><0。  



---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

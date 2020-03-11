---
layout: post
title: "Numerical Optimization Note Chapter 2"
description: "Note of Numerical Optimization"
categories: [Numerical Optimization]
tags: [Math]
redirect_from:
  - /2020/03/10/
---

# Numerical Optimization Exercise Chapter 2  

## Introduction  

本来我以为看吴立德教授的课就可以了，没想到他讲的实在是有点混乱，需要用大量的时间理解和整合，所以还是要自己结合大黄书梳理一遍记个笔记，不然看过之后根本没法回头重新回顾。  

笔记从第三章开始，前两章主要讲了一些整体性的知识，属于提纲挈领的内容，这里就略过不表了。这里由于github pages的插入公式实在是太不友好，我就尽量放图片，一些简单的公式就手打了，应该也看得懂。  

第三章介绍线搜索，内容还是很多的。所谓的线搜索，也叫做一维搜索，是最优化中的一种基础方法，它的主要步骤可以用一个公式来概括，即x<sub>k+1</sub> = x<sub>k</sub> + α<sub>k</sub>p<sub>k</sub>。在这个公式中，α<sub>k</sub>代表步长，p<sub>k</sub>代表方向，每一个迭代过程中x都会向着优化方向移动指定的步长，直到达到局部最优。这里线搜索需要保证局部最小值存在在自己指定的范围之中，也就是左右界必须确定才行。  

线搜索其实是一个广义的概念，大家一般还是称呼线搜索的一些子方法，比如牛顿法、最速下降法等等。这里不同的方法的方向选择不同，对于最速下降法（反梯度法），p<sub>k</sub><sup>G</sup> = -▽f(x<sub>k</sub>)，对于牛顿法，p<sub>k</sub><sup>N</sup> = -▽²f(x<sub>k</sub>)<sup>-1</sup>·▽f(x<sub>k</sub>)，对于拟牛顿法，p<sub>k</sub><sup>B</sup> = -B<sub>k</sub><sup>-1</sup>▽f(x<sub>k</sub>)，此外吴立德教授还提到了共轭梯度法，不过没有说具体形式，只是说后面会讲到，那么这里就不提了。  

## p<sub>k</sub>的选择  

对于p<sub>k</sub>的选择，不同的方法有着不同的选择方式。首先定义ψ(α)=f(x<sub>k</sub>+αp<sub>k</sub>)，此函数是以α为自变量，p<sub>k</sub>和x<sub>k</sub>都是已知的值，那么α的取值应该是ψ的全局最优解。  

对于该函数，很明显有ψ(0)=f(x<sub>k</sub>)，简写为f<sub>k</sub>, ψ'(α) = ▽f(x<sub>k</sub>+αp<sub>k</sub>)<sup>T</sup>p<sub>k</sub>, ψ'(0)=▽f<sub>k</sub><sup>T</sup>p<sub>k</sub>, p<sub>k</sub>是下降方向，所以ψ'(0)需小于0，即▽f<sub>k</sub><sup>T</sup>p<sub>k</sub><0。  
使用最速下降法的时候，p<sub>k</sub><sup>G</sup>=-▽f<sub>k</sub><sup>T</sup>，则ψ'(0)=p<sub>k</sub><sup>G</sup>▽f<sub>k</sub><sup>T</sup><0恒成立。  

使用牛顿法时，p<sub>k</sub><sup>N</sup> = -▽²f<sub>k</sub><sup>-1</sup>▽f<sub>k</sub>，则ψ'(0)=-▽f<sub>k</sub><sup>T</sup>▽f<sub>k</sub><sup>-1</sup>▽f<sub>k</sub>，当▽²f<sub>k</sub>正定时下降。  

使用拟牛顿法时，p<sub>k</sub><sup>B</sup> = -B<sub>k</sub><sup>-1</sup>▽f<sub>k</sub>，当B<sub>k</sub>正定时下降。  

## α<sub>k</sub>的选择  

对于步长的选择，我们有之前的函数ψ(α)=f(x<sub>k</sub>+αp<sub>k</sub>)，那么α<sub>k</sub> = argmin ψ(α) = argmin f(x<sub>k</sub>+αp<sub>k</sub>)。  

### Wolfe condition  

一般的，我们认为对于步长而言，一般需要提供足够的目标函数值的减少，也就是说每一次迭代都必须有足够明显的效果，那么我们可以显示指定一个值，使得每一步迭代后的下降值都必须大于该值才可将迭代过程视为有效。这就是Wolfe条件的第一个子条件，也叫做Armijo condition: f(x<sub>k</sub>+αp<sub>k</sub>) ≤ f(x<sub>k</sub>) + c<sub>1</sub>α▽f<sub>k</sub><sup>T</sup>p<sub>k</sub>, c<sub>1</sub>∈(0, 1)。  

该公式需要用图像来理解，下图是黄书里的示例图，l(α)代表上述不等式的右侧部分，只有ψ(α)小于l(α)的时候，α才能够被接受。  

![NB3-1](/images/Numerical-Optimization/NB3-1.png)  

第一个子条件可以保证下降的值不会太小，但是从上图中可以看出，当α非常小的时候，几乎是必然满足该子条件的，那么该迭代的效率就会很低，所以还需要第二个约束条件，使得每一次跌打α的值变化不能太小，这叫做curvature condition： ▽f(x<sub>k</sub>+α<sub>k</sub>p<sub>k</sub>)<sup>T</sup>p<sub>k</sub>≥c<sub>2</sub>▽f<sub>k</sub><sup>T</sup>p<sub>k</sub>, c<sub>2</sub>∈(c1, 1)，该约束左侧其实就是ψ'(α<sub>k</sub>)，其几何意义是使得ψ在α<sub>k</sub>点的斜率必须比c<sub>2</sub>倍的ψ'(0)大，下图是该约束的几何表示，我们可以看出在约束1的条件下又通过一阶导进行了第二次筛选。![NB3-2](/images/Numerical-Optimization/NB3-2.png)  

整体的筛选流程见下图：  
![NB3-3](/images/Numerical-Optimization/NB3-3.png)  

此外，Wolfe条件并不能保证找到的步长能够特别接近ψ的最小值，所以可以进一步调整条件2，从而使得α<sub>k</sub>在至少一个局部最小值或者驻点的宽邻域内，修改后的条件称作Strong Wolfe condition:  
f(x<sub>k</sub>+αp<sub>k</sub>) ≤ f(x<sub>k</sub>) + c<sub>k</sub>α<sub>k</sub>▽f<sub>k</sub><sup>T</sup>p<sub>k</sub>   (1)  
|▽f(x<sub>k</sub>+α<sub>k</sub>p<sub>k</sub>)<sup>T</sup>p<sub>k</sub>| ≤ c<sub>2</sub>|▽f<sub>k</sub><sup>T</sup>p<sub>k</sub>|, 0<c1<c2<1    (2)  

对于强Wolfe条件为什么能够保证在驻点或极小值点的邻域上，黄书上认为这样ψ'(α<sub>k</sub>)就不再会too positive，借此排除所有远离ψ的驻点的值。  

这里还有一个引理Lemma 3.1，这个引理简单来讲就是证明了满足Wolfe条件的α是存在的，也就是这个方法的可行性。具体的引理内容和证明过程见下图。  
![NB3-4](/images/Numerical-Optimization/NB3-4.png)  

此外黄书上还介绍了goldstein条件，不过吴老师没讲，我这里也先略过了，以后用得到再补好了。  

### α<sub>k</sub>的求解办法  

这里吴教授还介绍了一种求解φ'(α)=0的方法，称作函数折值法，其具体操作步骤是，给定β个ψ(α<sub>k</sub>)和ψ'(α<sub>k</sub>)值，求解β个多项式参数，从而接触ψ(α)的多项式形式。  

对于一般的情况，通常是先取任意一个α<sub>i</sub>，得到ψ(0), ψ'(0), ψ(α<sub>i</sub>), ψ'(α<sub>i</sub>)的值，从而解出一个ψ(α)=a<sub>0</sub>+a<sub>1</sub>x+a<sub>2</sub>x²+a<sub>3</sub>x<sup>3</sup>的参数解。  

## 收敛性和收敛速度  

### 一些概念  

在进入这一部分之前，需要先介绍一些概念，从而有利于后续部分的理解。  

#### 算法的收敛性  

所谓算法的收敛性，也就是目标函数f能不能收敛，其严格的定义为lim<sub>k→∞</sub>||▽f(x<sub>k</sub>)||=0. 此外还有一个略弱的表示，为lim<sub>k→∞</sub>inf||▽f(x<sub>k</sub>)||=0，也就是使得f梯度的下极限为0，换言之，就是使得||▽f<sub>k</sub>||始终小于一个足够小的值ε。满足这两个条件中的任何一个的算法，都可以视为收敛。  

#### 算法的收敛速度  

所谓的算法收敛速度，指的就是在每一轮迭代后x改变的程度，这里举了三个收敛方式，其收敛速度也各不相同。  

线性： ||x<sub>k+1</sub> - x<sup>*</sup>|| ≤ r·||x<sub>k</sub>-x<sup>*</sup>||, 0<r<1  
二次： ||x<sub>k+1</sub> - x<sup>*</sup>|| ≤ L·||x<sub>k</sub>-x<sup>*</sup>||², L>0  
超线性：||x<sub>k+1</sub> - x<sup>*</sup>|| ≤ o(||x<sub>k</sub>-x<sup>*</sup>||)  

#### 所需条件  

这一部分讲一下研究收敛性和收敛速度所需的一些基本条件。  

##### 关于f(x)  

首先定义水平集: L={x\|f(x)≤f(x<sub>0</sub>)}, 这里的x<sub>0</sub>为出发点，也就是说不管怎么优化，优化后的目标函数值肯定要比初始值要小，才算有效果。  

对于f(x)而言，一般有三个要求：f(x)有界（有上界且有下界），f(x)连续，f(x) Lipschitz连续: ||f(x)-f(y)||≤L||x-y||。  

对于▽f(x)而言，一般也有三个同样的要求：▽f(x)有界、连续且Lipschitz连续。对于二阶导▽²f(x)而言，也有同样的三个要求。那么对于f(x)，▽f(x)和▽²f(x)这三组不同的要求，其要求难度是逐渐升高的，要求阶数越高的导数，难度越大。  

##### 关于方向  

方向就是p<sub>k</sub>，这里上面已经说的很清楚了，需要使得▽f<sub>k</sub>²p<sub>k</sub><0, 那么最速下降法、牛顿法和拟牛顿法都有不同的要求，上面都提过了，这里就不再重复。  

##### 关于步长  

步长就是α<sub>k</sub>， α<sub>k</sub> = argmin f(x<sub>k</sub>+αp<sub>k</sub>)，上面也提到过，一般需要满足Wolfe条件。  

##### 关于x<sub>k</sub>  

设最优解为x<sup>\*</sup>，且最优解有二阶导数，那么要使得x<sub>k</sub>→x<sup>\*</sup>的充分条件就是▽f(x<sup>\*</sup>)=0，且▽²f(x<sup>\*</sup>)正定。  

### 收敛性  

由于在线搜索中，需要同时定义步长和方向，所以两者要一起分析才行。对于收敛速度而言，通常定义一个夹角θ<sub>k</sub>，用来表示搜索方向p<sub>k</sub>和最速下降方向-▽f<sub>k</sub>之间的角度。于是就有了下一个定理，也叫做Lipschitz condition:  
s若f(x)下面有界(f(x)≥M)，f(x)有一阶导数(▽f(x)存在), ||▽f(x) - ▽f(x<sub>avg</sub>)||≤L||x-x<sub>avg</sub>||, 则∑||▽f(x)||²cos²θ<sub>k</sub> < ∞  






---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

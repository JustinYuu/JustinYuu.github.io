---
layout: post
title: "Numerical Optimization Note Chapter 3"
description: "Note of Numerical Optimization"
categories: [Numerical Optimization]
tags: [Math]
redirect_from:
  - /2020/03/10/
---

# Numerical Optimization Note Chapter 3  

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

所谓算法的收敛性，也就是目标函数f能不能收敛，其严格的定义为lim<sub>k→∞</sub>\|\|▽f(x<sub>k</sub>)\|\|=0. 此外还有一个略弱的表示，为lim<sub>k→∞</sub>inf\|\|▽f(x<sub>k</sub>)\|\|=0，也就是使得f梯度的下极限为0，换言之，就是使得\|\|▽f<sub>k</sub>\|\|始终小于一个足够小的值ε。满足这两个条件中的任何一个的算法，都可以视为收敛。  

#### 算法的收敛速度  

所谓的算法收敛速度，指的就是在每一轮迭代后x改变的程度，这里举了三个收敛方式，其收敛速度也各不相同。  

线性： \|\|x<sub>k+1</sub> - x<sup>*</sup>\|\| ≤ r·\|\|x<sub>k</sub>-x<sup>*</sup>\|\|, 0<r<1  
二次： \|\|x<sub>k+1</sub> - x<sup>*</sup>\|\| ≤ L·\|\|x<sub>k</sub>-x<sup>*</sup>\|\|², L>0  
超线性：\|\|x<sub>k+1</sub> - x<sup>*</sup>\|\| ≤ o(\|\|x<sub>k</sub>-x<sup>*</sup>\|\|)  

#### 所需条件  

这一部分讲一下研究收敛性和收敛速度所需的一些基本条件。  

##### 关于f(x)  

首先定义水平集: L={x\|f(x)≤f(x<sub>0</sub>)}, 这里的x<sub>0</sub>为出发点，也就是说不管怎么优化，优化后的目标函数值肯定要比初始值要小，才算有效果。  

对于f(x)而言，一般有三个要求：f(x)有界（有上界且有下界），f(x)连续，f(x) Lipschitz连续: \|\|f(x)-f(y)\|\|≤L\|\|x-y\|\|。  

对于▽f(x)而言，一般也有三个同样的要求：▽f(x)有界、连续且Lipschitz连续。对于二阶导▽²f(x)而言，也有同样的三个要求。那么对于f(x)，▽f(x)和▽²f(x)这三组不同的要求，其要求难度是逐渐升高的，要求阶数越高的导数，难度越大。  

##### 关于方向  

方向就是p<sub>k</sub>，这里上面已经说的很清楚了，需要使得▽f<sub>k</sub>²p<sub>k</sub><0, 那么最速下降法、牛顿法和拟牛顿法都有不同的要求，上面都提过了，这里就不再重复。  

##### 关于步长  

步长就是α<sub>k</sub>， α<sub>k</sub> = argmin f(x<sub>k</sub>+αp<sub>k</sub>)，上面也提到过，一般需要满足Wolfe条件。  

##### 关于x<sub>k</sub>  

设最优解为x<sup>\*</sup>，那么要使得x<sub>k</sub>→x<sup>\*</sup>，需要满足两阶充分条件，即▽f(x<sup>\*</sup>)=0，且▽²f(x<sup>\*</sup>)正定。  

### 收敛性  

#### 定理3.2  

由于在线搜索中，需要同时定义步长和方向，所以两者要一起分析才行。对于收敛速度而言，通常定义一个夹角θ<sub>k</sub>，用来表示搜索方向p<sub>k</sub>和最速下降方向-▽f<sub>k</sub>之间的角度。于是就有了定理3.2，也叫做Zoutendijk Theorem:  
若f(x)下面有界(f(x)≥M)，f(x)有一阶导数(▽f(x)存在)且满足Lipschitz条件(\|\|▽f(x) - ▽f(x<sub>avg</sub>)\|\|≤L\|\|x-x<sub>avg</sub>\|\|), α<sub>k</sub>满足Wolfe条件，p<sub>k</sub>下降，则∑\|\|▽f(x)\|\|²cos²θ<sub>k</sub> < ∞  

下面是对该定理的证明：  

由Wolfe条件1，也就是f<sub>k+1</sub> ≤ f<sub>k</sub> + c<sub>1</sub>α<sub>k</sub>▽f<sub>k</sub><sup>T</sup>p<sub>k</sub>可以得到  
f<sub>k</sub> - f<sub>k+1</sub> ≥ -c<sub>1</sub>α<sub>k</sub>▽f<sub>k</sub><sup>T</sup>p<sub>k</sub> = c<sub>1</sub>α<sub>k</sub>\|\|▽f<sub>k</sub>\|\|·\|\|p<sub>k</sub>\|\|·cosθ<sub>k</sub>  

因为α<sub>k</sub>>0, 所以  
f<sub>k</sub> - f<sub>k+1</sub> / α<sub>k</sub>·\|\|p<sub>k</sub>\|\| ≥ c<sub>1</sub>\|\|▽f<sub>k</sub>\|\|cosθ<sub>k</sub>  (1)  

对于该式子的理解是这样的，由于α<sub>k</sub>\|\|p<sub>k</sub>\|\| = \|\|x<sub>k+1</sub> - x<sub>k</sub>\|\|，所以上述等式左侧部分为x<sub>k+1</sub>和x<sub>k</sub>之间的平均斜率(的负值)。  

由Wolfe条件2，也就是▽f<sub>k+1</sub><sup>T</sup>p<sub>k</sub> ≥ c<sub>2</sub>▽f<sub>k</sub><sup>T</sup>p<sub>k</sub>，有  

▽f<sub>k+1</sub><sup>T</sup>p<sub>k</sub> - ▽f<sub>k</sub><sup>T</sup>p<sub>k</sub> ≥ (c<sub>2</sub>-1)▽f<sub>k</sub><sup>T</sup>p<sub>k</sub>  

即(▽f<sub>k+1</sub> - ▽f<sub>k</sub>)<sup>T</sup>p<sub>k</sub> ≥ (1-c<sub>2</sub>)(-▽f<sub>k</sub><sup>T</sup>p<sub>k</sub>)  (2)  

由Lipschitz条件得 \|(▽f<sub>k+1</sub> - ▽f<sub>k</sub>)<sup>T</sup>p<sub>k</sub>\| ≤ \|\|▽f<sub>k+1</sub> - ▽f<sub>k</sub>\|\|·\|\|p<sub>k</sub>\|\| ≤ L·\|\|x<sub>k+1</sub> - x<sub>k</sub>\|\|·\|\|p<sub>k</sub>\|\| = L·α<sub>k</sub>\|\|p<sub>k</sub>\|\|²  (3)  

由(2)(3)得，  

(1-c<sub>2</sub>)(-▽f<sub>k</sub>p<sub>k</sub>) = (1-c<sub>2</sub>)·\|\|▽f<sub>k</sub>\|\|·\|\|p<sub>k</sub>\|\|cosθ<sub>k</sub>≤α<sub>k</sub>\|\|p<sub>k</sub>\|\|  (4)  

由(1)(4)得，  

f<sub>k</sub> - f<sub>k+1</sub> ≥ c<sub>1</sub>(1-c<sub>2</sub>)/L * \|\|▽f<sub>k</sub>\|\|²cos²θ<sub>k</sub>  

由于f有下界，则∑<sub>k</sub> c<sub>1</sub>(1-c<sub>2</sub>)/L \|\|▽f<sub>k</sub>\|\|²cos²θ<sub>k</sub> ≤ ∑<sub>k</sub> f<sub>k</sub>- f<sub>k+1</sub> < ∞  

#### 补充知识  

这里还需要一些矩阵的补充知识，若A正定，则A=UΛU<sup>T</sup>, 其中Λ为元素值为A的特征值的对角矩阵，则K(A) = λ<sub>max</sub>/λ<sub>min</sub>。且由于A正定，那么A<sup>-1</sup>存在，A<sup>-1</sup>的特征值为1/λ<sub>1</sub>, 1/λ<sub>2</sub>, ..., 1/λ<sub>n</sub>, 同理A<sup>1/2</sup>的特征值为√λ<sub>1</sub>, √λ<sub>2</sub>, ..., √λ<sub>n</sub>。  

#### 最速下降法  

对于最速下降法，p<sub>k</sub> = -▽f<sub>k</sub>, cosθ<sub>k</sub> = 1, θ<sub>k</sub> = 0°。由定理3.2可知∑<sub>k</sub>\|\|▽f<sub>k</sub>\|\|²cos²θ<sub>k</sub> = ∑<sub>k</sub>\|\|▽f<sub>k</sub>\|\|²＜∞，故f<sub>k</sub>收敛，且\|\|▽<sub>k</sub>\|\|→0  

#### 拟牛顿法和牛顿法  

对于拟牛顿法，B<sub>k</sub>正定，这里由于cosθ不为1，所以我们要想证明函数收敛，必须证明lim<sub>k→∞</sub>cos²θ<sub>k</sub> ≥ δ > 0, 则f<sub>k</sub>收敛。  

若上述条件满足，则需满足\|\|B<sub>k</sub>\|\|·\|\|B<sub>k</sub><sup>-1</sup>\|\|， 也就是K(B<sub>k</sub>)，即B<sub>k</sub>的条件数，需要小于等于M，即不是无穷大。  

如果满足上述条件，那么cosθ<sub>k</sub> = -▽f<sub>k</sub> · p<sub>k</sub>/ (\|\|▽f<sub>k</sub>)\|\| · \|\|p<sub>k</sub>\|\| = ▽f<sub>k</sub> B<sub>k</sub><sup>-1</sup> ▽f<sub>k</sub> / (\|\|▽f<sub>k</sub>\|\|·\|\|B<sub>k</sub><sup>-1</sup>▽f<sub>k</sub>) ≥ (1/λ<sub>n</sub>)·\|\|▽f<sub>k</sub>\|\|² / (\|\|B<sub>k</sub><sup>-1</sup>\|\|·\|\|▽f<sub>k</sub>\|\|²) =(1/λ<sub>n</sub>)/(1/λ<sub>1</sub>) = λ<sub>1</sub> / λ<sub>n</sub> ≥ 1/M > 0  

对于牛顿法，B<sub>k</sub>为Hessian矩阵，不过吴老师没有进行下一步的阐述。  

### 收敛速度  

收敛速度要根据方法不同来讨论。  

#### 最速下降法  

对于最速下降法，收敛速度是线性的，这里老师没有证明。  

#### 牛顿法  

牛顿法就比较复杂了，对于最纯粹的牛顿法，一般先将α<sub>k</sub>定义为1，那么x<sub>k+1</sub> = x<sub>k</sub> - ▽²f<sub>k</sub><sup>-1</sup> · ▽f<sub>k</sub>。对于这种情况，有定理3.5：假设f有下界，▽²f(x)在x<sup>\*</sup>的邻域Lipschitz连续，在x<sup>\*</sup>满足二阶充分条件▽f(x<sup>\*</sup>)=0, ▽²f(x)正定，且x离x<sup>\*</sup>充分接近，则有:  

1. x<sub>k</sub> → x<sup>\*</sup>  
2. \|\|x<sub>k+1</sub> - x<sup>*</sup>\|\| ≤ L<sub>1</sub> \|\|x<sub>k</sub>-x<sup>\*</sup>\|\|² （二次收敛）  
3. \|\|▽f<sub>k+1</sub> ≤ L<sub>2</sub>\|\|▽f<sub>k</sub>\|\|² （二次收敛）  

下面是证明，这里采用的是吴教授的中值定理证明方式，和黄书上的定积分中值定理证明方式略有不同。  

首先证明2. \|\|x<sub>k+1</sub> - x<sup>*</sup>\|\|  
= \|\|x<sub>k+1</sub> -▽²f<sub>k</sub><sup>-1</sup>▽f<sub>k</sub> - x<sup>*</sup>\|\|  
= \|\|x<sub>k+1</sub> - x<sup>*</sup> - ▽²f<sub>k</sub><sup>-1</sup>▽f<sub>k</sub>\|\|  
= \|▽²f<sub>k</sub><sup>-1</sup>\[▽²f<sub>k</sub>(x<sub>k</sub>-x<sup>\*</sup>) - (▽f<sub>k</sub>-▽f<sub>\*</sub>)]\|  
≤ \|\|▽²f<sub>k</sub><sup>-1</sup>\|\| · \|\|▽²f<sub>k</sub>(x<sub>k</sub>-x<sup>\*</sup>) - ▽²f(x<sub>k</sub>+t(x<sub>k</sub>-x<sup>*</sup>))(x<sub>k</sub>-x<sup>\*</sup>)\|\|, t∈(0, 1)  
= \|\|▽²f<sub>k</sub><sup>-1</sup>\|\| · \|\|(▽²f<sub>k</sub> - ▽²f(x<sub>k</sub>+t(x<sub>k</sub> - x<sup>\*</sup>)))\|\| · \|\| x<sub>k</sub> - x<sup>\*</sup>\|\|  
≤(Lipschitz) \|\|▽²f<sub>k</sub><sup>-1</sup>\|\|·L·\|\|t(x<sub>k</sub>-x<sup>\*</sup>)\|\|·\|\|x-x<sup>\*</sup>\|\|  
≤ \|\|▽²f<sub>k</sub><sup>-1</sup>\|\|·L·\|\|x-x<sup>\*</sup>\|\|² ≤ 2\|\|▽²f(x<sup>\*</sup>)<sup>-1</sup>\|\|·L·\|\|x-x<sup>\*</sup>\|\|²(x和x<sup>*</sup>足够接近)  

由于L前面的部分为常数，所以二次收敛证毕。又由于2成立，那么\|\|x<sub>k+1</sub>-x<sup>\*</sup>\|\|可收敛到0，故1也成立，下面证3。  

同样有中值定理得到，▽f(x<sub>k</sub>) - ▽f(x<sup>\*</sup>) = ▽²f(x<sub>k</sub>+t(x<sub>k</sub>-x<sup>\*</sup>))(x<sub>k</sub>-x<sup>\*</sup>)，又由于牛顿法中，▽²f(x<sub>k</sub>)p<sub>k</sub> = -▽f(x<sub>k</sub>)，故▽²f<sub>k</sub>p<sub>k</sub>+▽f<sub>k</sub> = 0.  

那么我们有\|\|▽f<sub>k+1</sub>\|\| = \|\|▽f<sub>k+1</sub> - ▽²f<sub>k</sub>p<sub>k</sub> - ▽f<sub>k</sub>\|\| =  
\|\|▽f<sub>k+1</sub> - ▽f<sub>k</sub> - ▽²f<sub>k</sub>p<sub>k</sub>\|\| ≤  \|\|▽²f(x<sub>k</sub>+tp<sub>k</sub>)·(x<sub>k+1</sub> - x<sub>k</sub>) - ▽²f<sub>k</sub>p<sub>k</sub>\|\|  
= \|\|(▽²f(x<sub>k</sub>+tp<sub>k</sub>)-▽²f<sub>k</sub>)p<sub>k</sub>\|\| ≤ L·\|\|tp<sub>k</sub>\|\|·\|\|p<sub>k</sub>\|\|  
≤ L·\|\|p<sub>k</sub>\|\|² = L·\|\|▽²f<sub>k</sub><sup>-1</sup>▽f<sub>k</sub>\|\|²  
≤ L·\|\|▽²f<sub>k</sub><sup>-1</sup>\|\|²\|\|▽f<sub>k</sub>²\|\| ≤ L·2·\|\|▽²f(x<sup>\*</sup><sup>-1</sup>)\|\|²\|\|▽f<sub>x</sub>\|\|²  

前面部分为常数，故得证。  

#### 修正的牛顿法  

标准的牛顿法有几个条件，▽f<sub>k</sub>默认是正定的，且α<sub>k</sub>=1满足Wolfe条件，但是在实际情况下这两点可能不会默认满足，所以这时候需要进行一定的改进。  

对于α<sub>k</sub>不符合Wolfe条件的情况，只需要将α<sub>k</sub>放缩到符合的情况即可，这里吴教授简单略过了。  

对于二阶导不正定的情况，其实就是因为▽²f<sub>k</sub>=UΛU<sup>T</sup>中Λ的特征值不全大于0。这里提出了两种方法，第一种就是把负值填充成正值，也就是把▽²f<sub>k</sub> + E<sub>k</sub> = B<sub>k</sub>, Ek中的Λ'的值为需要补全的特征值，在需要补全的位置是某一正值，不需要补全的位置为0，正值的具体值为λ'<sub>i</sub> = ε - λ<sub>i</sub>, ε为指定的一个正值。  

实际情况中一般使用另一个方法，因为第一种方法太麻烦了，需要对于每一个负特征值设定一个补全值，所以一般使用B<sub>k</sub> = ▽²f<sub>k</sub> + λI的方法，即把原矩阵统一加一个单位矩阵的λ倍，从而使得所有的值都是正的，这种方法虽然对所有的值都进行了修改，但是操作起来非常简单，只需要设定一个数值就可以了。  

在实际的牛顿法迭代中，一般含有多个迭代过程，首先对k进行迭代，在这一轮迭代中，又要对α和λ进行迭代，那么对λ迭代的过程中，需要采用数值法的方式来快速的判定矩阵是否正定，这里一般采用矩阵Cholesky分解方式。  

Cholesky分解需要通过一个定理完成：如果A是一个共轭正定阵，则A=LDL<sup>T</sup>，L为一个下三角阵，D为一个对角阵，且对角阵里的对角元素均大于0。下面举一个n=3的例子，未知数是矩阵L中的L<sub>21</sub>, L<sub>31</sub>, l<sub>32</sub>, 对角阵的d<sub>1</sub>, d<sub>2</sub>, d<sub>3</sub>，已知的是矩阵A中的值，那么有：  
a<sub>11</sub>=d<sub>1</sub>, a<sub>21</sub> = l<sub>21</sub>d<sub>1</sub>, a<sub>31</sub> = l<sub>31</sub>d<sub>1</sub>  
a<sub>22</sub> = d<sub>1</sub>l<sub>21</sub>l<sub>21</sub> + d<sub>2</sub>, a<sub>32</sub> = b<sub>1</sub>l<sub>31</sub>l<sub>31</sub> + d<sub>2</sub>l<sub>32</sub>  
a<sub>33</sub> = d<sub>1</sub>l<sub>31</sub>l<sub>31</sub> + d<sub>2</sub>l<sub>32</sub>l<sub>32</sub> + d<sub>3</sub>  

解得d<sub>1</sub> = a<sub>11</sub>, l<sub>21</sub> = a<sub>21</sub> / d<sub>1</sub>, l<sub>31</sub> = a<sub>31</sub> / d<sub>1</sub>, d<sub>2</sub> = a<sub>22</sub> - d<sub>1</sub>l<sub>21</sub>l<sub>21</sub>  
l<sub>32</sub> = (a<sub>22</sub> - b<sub>1</sub>l<sub>31</sub>l<sub>31</sub>)/d<sub>2</sub>, d<sub>3</sub> = a<sub>33</sub> - d<sub>1</sub>l<sub>31</sub>l<sub>31</sub> - d<sub>2</sub>l<sub>32</sub>l<sub>32</sub>  
从而完成对于共轭正定矩阵A的cholesky分解。  

将n扩展到一般情况，那么整个cholesky分解的算法可写成以下形式：  

for j = 1,2,...,n:  
    d<sub>j</sub> = a<sub>jj</sub> - ∑d<sub>s</sub>l<sub>js</sub> (d<sub>j</sub>>0)  
    for i = j+1, j+2, ...,n:  
      l<sub>ij</sub> = (a<sub>ij</sub> - ∑d<sub>s</sub>l<sub>js</sub>l<sub>js</sub>) / d<sub>j</sub>  

整体的流程非常简单，复杂度为O(n³)，虽然复杂度看起来很高，但是这种方法求得的数值解比一般的高斯消去法速度更快。  

那么总结一下，对于改进的牛顿法的x<sub>k</sub>的一轮更新步骤，首先要计算▽f<sub>k</sub>，▽²f<sub>k</sub>，对于τ = τ<sub>1</sub>, τ<sub>2</sub>,... , 进行▽²f<sub>k</sub>+τI的cholesky分解，直到正定。接下来将▽²f<sub>k</sub>+τI看做L<sup>T</sup>DL，解L<sup>T</sup>DLp<sub>k</sub> = -▽f<sub>k</sub>， 解得p<sub>k</sub>，搜索α<sub>k</sub>步长，满足Wolfe条件，得到α<sub>k</sub>，最后可以得到x<sub>k+1</sub> = x<sub>k</sub> + α<sub>k</sub>p<sub>k</sub>。  

## 总结  

这一章终于学习完毕了，内容是真的多，不过吴教授已经提前给我做了心理准备，第四章会更难的:)  

---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

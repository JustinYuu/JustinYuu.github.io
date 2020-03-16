---
layout: post
title: "Numerical Optimization Note Chapter 4"
description: "Note of Numerical Optimization"
categories: [Numerical Optimization]
tags: [Math]
redirect_from:
  - /2020/03/16/
---

# Numerical Optimization Note Chapter 4  

## Introduction  

第四章讲的叫做信赖域方法(Trust Region Method)，所谓信赖域算法，就是相对于线搜索方法而言的另外一种优化算法，其主要思想是设置一个阈值△<sub>k</sub>，使得球B(x<sub>k</sub>, △<sub>k</sub>) = {x\| \|\|x-x<sub>k</sub>\|\| ≤ △<sub>k</sub>}为分析x<sub>k</sub>的信赖域。在信赖域中，用泰勒二阶展开来近似等于f的值，即  
f(x<sub>k</sub>+p) ≈ m<sub>k</sub>(p) = f(x<sub>k</sub>) + ▽f(x<sub>k</sub>)<sup>T</sup>p + 1/2 p<sup>T</sup>B<sub>k</sub>p  
▽f(x<sub>k</sub>+p) ≈ ▽m<sub>k</sub>(p) = ▽f(x<sub>k</sub>) + B<sub>k</sub>p  

B<sub>k</sub> = ▽²f(x<sub>k</sub>)的时候，形式是严格的泰勒展开，这时候是效果最优秀的，但是实际应用过程中，由于计算太复杂，所以一般使用简单方法来代替。  

若求f(x)的最小值，这里的方式是求m<sub>k</sub>(p)的最小值，那么有子问题: minm<sub>k</sub>(p) = f<sub>k</sub> + ▽f<sub>k</sub><sup>T</sup>p + 1/2 p<sup>T</sup>B<sub>k</sub>p, \|\|p\|\|≤△<sub>k</sub>, p<sub>k</sub> = argmin m<sub>k</sub>(p)，从而确定p的值。  

然而，由于p<sub>k</sub>是由m<sub>k</sub>选出的，而不是直接由f<sub>k</sub>选出的，那么当m<sub>k</sub>和f<sub>k</sub>的逼近不好时，结果有可能并不理想，因此作者引入了一个新的参数ρ<sub>k</sub> = (f(x<sub>k</sub>) - f(x<sub>k</sub>+p<sub>k</sub>))/(m<sub>k</sub>(o)-m<sub>k</sub>(p<sub>k</sub>))，从而衡量m<sub>k</sub>和f<sub>k</sub>的逼近程度，显然ρ=1的时候最好。那么随着ρ的值不同，整个算法的更新规则也有不同，具体如下：  

ρ<sub>k</sub> < 1/4时，△<sub>k+1</sub> = 1/4 △<sub>k</sub>， ρ<sub>k</sub>＞3/4且\|\|p<sub>k</sub>\|\|=△<sub>k</sub>，△<sub>k+1</sub> = 2△<sub>k</sub>。  
p<sub>k</sub><η∈(0, 1/4)时，x<sub>k+1</sub>=x<sub>k</sub>，也就是不更新。否则x<sub>k+1</sub>=x<sub>k</sub>+p<sub>k</sub>，也就是更新。  





---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

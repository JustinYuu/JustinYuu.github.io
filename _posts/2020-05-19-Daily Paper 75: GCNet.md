---
layout: post
title: "Daily Paper 75: GCNet"
description: "Notes"
categories: [CV-Attention]
tags: [Paper]
redirect_from:
  - /2020/05/19/
---

# Daily Paper 75: GCNet: Non-local Networks Meet Squeeze-Excitation Networks and Beyond  

## Introduction  

这篇文章真是好东西，是大清和港科大几个在MSRA的实习生写的。从名字就可以看出来，这篇文章主要讨论的是Non-local Networks和SENet的对比与结合。昨天刚看完Non-local Networks，当时给我的直观感受就是方法偏暴力，考虑了太多没用的信息，导致增加了太多本不需要的计算复杂度。当然考虑了太多没用的信息这句话只是我的猜测，而这篇文章的作者真的去做了下实验，发现一张图片中不同位置的non-local模块得到的全局信息几乎是一样的。  

这就问题大了。如果真的是这样，那就说明non-local模块所引以为豪的pairwise信息实质上对于任意pair来说都是一样的，那学习到的长距离依赖性到底是否真的有用就值得怀疑了。在这篇paper中，作者利用了这个发现，基于一个与query无关的公式，在准确率和non-local network相当的前提下大幅降低了计算复杂度。而作者发现他们搞的这个简化版网络其实就是SE-Net。。。我猜这时候作者的内心一定是崩溃的，所以作者又做了一些微小的改动，按照作者的说法是把SE-Net和Non-local Network结合成了一个由三个步骤组成的用于全局语义建模的通用框架。在这个框架之内，作者设计了一个更好的模块，叫做global context block(GC block)。该模块更为轻巧，从而能够使得作者应用该模块去建立一个全局语义网络GCNet，并得到了比NLNet和SENet都优秀的表现。  










---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

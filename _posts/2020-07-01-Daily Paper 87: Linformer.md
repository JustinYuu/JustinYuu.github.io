---
layout: post
title: "Daily Paper 87: Linformer"
description: "Notes"
categories: [NLP]
tags: [Paper]
redirect_from:
  - /2020/07/01/
---

# Daily Paper 87: Linformer: Self-Attention with Linear Complexity  

## Introduction  

这篇文章是Facebook在六月中旬挂在arxiv上的，是对transformer在计算量方面的一种改进。作者认为虽然transformer的表现非常好，但是训练和部署这些模型需要花费相当多的时间，因为标准自注意力机制需要花费O(n²)的时间和空间复杂度。这里作者使用一个低秩的矩阵来近似标准自注意力机制，从而在时间和空间维度上把transformer的时间和空间复杂度变为O(n)。作者将改良后的transformer称为Linformer，在时间和空间复杂度大大降低的前提下取得了和标准transformer模型相当的性能。  

作者的灵感来源于发现自注意力是低秩的，更为确切的说，作者同时在理论和实验上证明了自注意力机制形成的随机矩阵可以被近似为一个低秩矩阵。因此作者就将原始的scaled dot-product attention分解为多个小的线性投影attention，这些操作的结合组成了一个对原始注意力的低秩因式分解。作者的复杂度对比如下:  
![87-1](/images/daily paper/87-1.png)  

## Method  

首先回顾一下transformer的操作，下面是多头注意力中某一头的公式:  
![87-2](/images/daily paper/87-2.png)  

其中P可以看作一个上下文映射矩阵，但是计算P计算量是比较大的，由于需要完成两个n×d矩阵的相乘，所以时间和空间复杂度都是O(n²)，因此对于序列长度的二次时间/空间复杂度也成为了Transformer的瓶颈。  

最近也有一些方法来解决这一问题，比如mixed precision，知识蒸馏，sparse attention和Locally-sensitive hashing attention。也有通过提升优化效率的方式来减轻计算负担的方式。作者认为这些方式大多采用时间换空间的方式，作者力图找到一种时间复杂度和空间复杂度同时降低的方法。  




---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

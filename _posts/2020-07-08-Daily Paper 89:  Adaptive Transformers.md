---
layout: post
title: "Daily Paper 89: Adaptive Transformers"
description: "Notes"
categories: [NLP]
tags: [Paper]
redirect_from:
  - /2020/07/08/
---

# Daily Paper 89: Adaptive Transformers for Learning Multimodal Representations  

## Introduction  

这篇文章发表在ACL2020 Student Research Workshop上，主要是对Transformer的改进。作者认为之前的transformer虽然有很广泛的应用，但是参数太多，需要巨大的计算量。因此作者对transformer进行了扩展，使用一些适应性方法来获得更多的模型可解释性和计算效率。具体来，作者研究了注意力的分散，稀疏和结构化的dropout方法，以帮助了解他们的注意力机制如何扩展到视觉和语言任务，此外作者还证明了这些方法能够使得其更了解网络是如何处理复杂的输入序列和不同模态之间的稀疏偏好的。  

总体来讲，这篇文章的贡献如下：首个即将自适应方法从语言特征扩展到多模态特征上，以研究它们如何对齐以捕获不同模态之间的复杂关系，并研究了对齐这些方法带来的影响，从而通过消融分析了解其兼容性；进行了可解释性分析，以了解这些方法如何增强人类对注意力行为和适应性方法的理解；提供了针对多模态输入序列的最新自适应方法的实验结果。总结一下就是搬了一个叫做自适应的方法过来，然后进行了可解释性分析，并得到了不错的实验结果。  

## Background  

由于使用的方法应该不是作者独创的，所以background这一环节其实就相当于之前文章的method部分了。这里主要对作者使用的backbone网络，自适应注意力跨度、自适应稀疏注意力两种自适应方法，以及一个trick LayerDrop进行了介绍，下面分别详细介绍这几种方法。  

### LXMERT  






---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

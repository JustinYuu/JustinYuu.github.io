---
layout: post
title: "Daily Paper 94: mixup"
description: "Notes"
categories: [CV-classic]
tags: [Paper]
redirect_from:
  - /2020/08/05/
---

# Daily Paper 94: Synthesizer: Rethinking Self-Attention in Transformer Models  

## Introduction  

今天看的这篇是MIT的张宏毅在FAIR实习的时候发在ICLR2018上的，主要贡献是提出了mixup这一非常好用的trick。作者的motivation在于他认为当前的大型神经网络虽然非常有效，但是仍然存在一系列缺陷，比如对于对抗样本的记忆性和敏感性。因此作者提出了mixup这一简单的学习方式，用来缓解这些问题。作者认为在本质上，mixup使用训练实例和其标签对的凸结合来训练神经网络，从而将神经网络正规化以支持训练样本间的简单线性行为。作者发现mixup在多个数据及上都能提高SOTA神经网络架构的泛化性能，并发现mixup能够减少网络对错误样本的记忆力，增加对对抗样本的鲁棒性，能够稳定GAN的训练过程。  




---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

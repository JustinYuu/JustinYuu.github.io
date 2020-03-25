---
layout: post
title: "Daily Paper 55: Video Action Transformer Network"
description: "Notes"
categories: [CV-Meta]
tags: [Paper]
redirect_from:
  - /2020/03/25/
---

# Daily Paper 55: Video Action Transformer Network  

## Introduction  

这篇文章是CMU、DeepMind和VGG组合作发表在CVPR2019上的，主要contribution是把transformer用于非小样本动作识别，实现了动作识别的STOA。这里的attention主要是用来结合特定的人的相关区域和所在的语境，从而识别动作。multi-head中的每一个head都用来计算一个clip embedding，用来着眼于当前动作下不同的部分，比如手、眼、其他人等。  

动作识别一直是一个很复杂的东西，除了要观察目标人物而外，还要观察其周围的环境，与之互动的人和物体等，这就需要通过注意力机制来进行两者之间的联系。作者自然而然的想到了Transformer，使用注意力机制来将上下文的联系考虑在内。作者的模型主要由以下几部分组成：一个时空I3D模型用来提取特征，一个区域提案网络RPN用来定位动作执行人，两者的结果作为Transformer的输入，整合其他人和环境的信息。transformer主要聚焦在人脸和手，由于这是进行动作的主要部位，一般会隐含更多的信息。这些东西都没有经过显式的监督，而是在动作分类的训练过程中获取。  



---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

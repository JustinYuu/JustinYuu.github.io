---
layout: post
title: "Improving Deep Neural Networks: Hyperparameter tuning, Regularization and Optimization Chapter 2"
description: "Notes"
categories: [Neural-Networks-and-Deep-Learning]
tags: [Python]
redirect_from:
  - /2019/03/13/
---

# Improving Deep Neural Networks: Hyperparameter tuning, Regularization and Optimization Chapter 2  

第二周开始啦，本周依然是继续学习一系列优化算法。  

## Optimization Algorithms  

这一章是本节的重点，其中包含了本章的几乎所有重点。  

首先介绍的是小批量梯度下降，这在之前machine-learning那门课程中介绍过，这是当初记的笔记：[Machine Learning by Andrew Ng Chapter 10](http://justin-yu.me/blog/2019/02/08/Machine-Learning-by-Andrew-Ng-Chapter-10/)。小批量梯度下降是介于批量梯度下降和随机梯度下降的中间状态，每过一定数量的数据就进行一次梯度下降。  
每一次学习梯度下降，我都会有一个疑问，梯度下降为什么不会陷入局部最优？我找了一篇知乎的文章，里面用了很多数学理论，我还无法理解，先放在这里以后慢慢理解[凸优化](https://zhuanlan.zhihu.com/p/30486793)，总的来说这是一个凸优化，所以极小值必定是最小值，但是为什么是凸优化需要严格的证明，很遗憾，证明过程我看不懂。据吴大大所讲，小批量梯度下降一般会朝着全局最优的方向下降，但是随机梯度下降有可能会朝着错误的方向进行，但是最终还是会朝着全局最优下降。但是我个人并不是很喜欢这种“通知”式的教学方式，也许证明会很复杂困难，但至少应该给出一些可选的链接来帮助学习，毕竟我还是不懂为什么最终会朝着全局最优下降。  
总的来说，如果训练集规模在2000以下，可以直接用批量梯度下降，但是如果在2000以上，可以选择m为64-512的batch来进行mini-batch梯度下降(2的整数幂)，batch的大小本身也是超参数之一。  

接下来学习了一个叫做指数加权平均的东东，其目的是使数据更为平均、平滑。操作方式是将前面的加权平均乘以一个参数θ，然后加上（1-θ）乘以这一层的值，得数就是指数加权平均值，事实证明θ取0.9时效果最好，接下来讲的几个优化函数也都是以此为基础进行优化的。此外还有一个小trick叫做偏差修正，与指数加权平均配合使用，可以更加精确的计算平均值。偏差修正实现起来非常简单，只需要把最后的结果除以（1-β）就可以，β就是之前提到的参数θ，因为1-β一般都比较小，那么在公式vt = βv(t-1)+(1-β)中，在一开始vt的结果将会非常小，以至于远远小于原本的数值，所以要进行这种偏差修正。   

紧接着介绍了一个利用指数加权平均的梯度下降法——Momentum(动量)梯度下降法，算法的主要思想是 计算梯度的指数加权平均，然后使用这个梯度来更新权重。值得注意的是在动量梯度下降的时候不需要进行偏差修正，此外，动量梯度下降的指数加权中，有人会选择将(1-β)这一个系数项舍去，也有人不会。如果舍去，学习率α将会在每次迭代后重新调整，这样可能不太直观，所以吴恩达倾向于不舍去，但是不管舍去与否，动量梯度下降的效果都会非常好。  

接下来学习了一个叫做RMSprop的方法，全称为Root Mean Square prop,中文名称为均方根传递，也可以加速梯度下降。这种方法在计算加权平均数时，将dW和db两项平方，然后再梯度下降的时候将dW和db除以加权平均数的平方根，同时为了防止分母为0，一般在除以平方根时加上一个特别小的epsilon，通常取10e-8. 在水平方向上，即W的方向上，我们希望学习速率较快 而在垂直方向上，即b的方向上 我们希望降低垂直方向上的振荡，对于S_dW和S_db这两项（即水平和数值方向上的均方根值） 我们希望S_dW相对较小，因此这里除以的是一个较小的数；而S_db相对较大，因此这里除以的是一个较大的数，这样就可以减缓垂直方向上的更新。  

Adam优化算法是将动量梯度下降和RMSprop结合在一起的算法。首先要讲Vdw,Sdw,Vdb,Sdb初始化为0，然后用小批量梯度下降计算出dW和db，然后分别用动量梯度下降和RMSprop计算一遍，分别除以(1-β1/β2)，然后进行梯度下降，不同点是用V除以S的平方根，也就是说将两个算法结合了起来。至于Adam这个名字的由来，并不是发现者的名字叫做Adam，而是因为Adam代表Adaptive Moment Estimation（自适应估计）。  

本章的最后，Andrew陈述了学习率衰减的必要性。随着小批量梯度下降的不断进行，如果不衰减学习率，那么很难达到真正的最优，总是因为步长太大而在最优旁边移动。衰减的方式有指数级衰减和离散型衰减等等。此外，吴恩达讲了一个我非常关心的问题：局部最优。事实上，在神经网络中，大部分斜率为0的点并不是局部最优点， ，而是鞍点（Saddle Point）。由于大部分情况下都在高维空间进行，那么如果是局部最优点，则需要在所有维度导数都是0且凹凸性相同，当然概率非常低，所以更容易碰到的就是虽然斜率为0但是凹凸性不完全相同的鞍点，而鞍点并不是局部最优。那么在鞍点上有没有什么坏处呢？答案是有的，会使训练到达一个停滞区，从而使学习速率显著变慢，那么我们之前所学的一系列优化算法也就是在优化这些学习速率变慢的情况。  

## Programming Assignment  
本周的作业相对来说量较大，分别实现了小批量梯度下降、Momentum和Adam优化，并分析了其优化性能。  

### mini-batch gradient descent  

    parameters["W" + str(l+1)] = parameters['W'+str(l+1)] - learning_rate * grads['dW' + str(l+1)]
    parameters["b" + str(l+1)] = parameters['b'+str(l+1)] - learning_rate * grads['db' + str(l+1)]  
		
---

	mini_batch_X = shuffled_X[:, k* mini_batch_size : (k+1) *  mini_batch_size]
	mini_batch_Y = shuffled_Y[:, k* mini_batch_size : (k+1) *  mini_batch_size]
	
	mini_batch_X = shuffled_X[:, num_complete_minibatches *  mini_batch_size:]
	mini_batch_Y = shuffled_Y[:, num_complete_minibatches *  mini_batch_size:]
	
---  

### Momentum  

	v["dW" + str(l+1)] = np.zeros(parameters["W"+str(l+1)].shape)
	v["db" + str(l+1)] = np.zeros(parameters["b"+str(l+1)].shape)

---  

	v["dW" + str(l+1)] = beta * v["dW"+str(l+1)] + (1-beta)* grads['dW'+str(l+1)]
	v["db" + str(l+1)] = beta * v["db"+str(l+1)] + (1-beta)* grads['db'+str(l+1)]
	parameters["W" + str(l+1)] = parameters["W"+str(l+1)] - learning_rate * v["dW"+str(l+1)]
	parameters["b" + str(l+1)] = parameters["b"+str(l+1)] - learning_rate * v["db"+str(l+1)]  
	
### Adam

	v["dW" + str(l+1)] = np.zeros(parameters["W" + str(l+1)].shape)
	v["db" + str(l+1)] = np.zeros(parameters["b" + str(l+1)].shape)
	s["dW" + str(l+1)] = np.zeros(parameters["W" + str(l+1)].shape)
	s["db" + str(l+1)] = np.zeros(parameters["b" + str(l+1)].shape)  
	
---  

	v["dW" + str(l+1)] = beta1 * v["dW" + str(l+1)] + (1-beta1)*grads['dW' + str(l+1)]
	v["db" + str(l+1)] = beta1 * v["db" + str(l+1)] + (1-beta1)*grads['db' + str(l+1)]
	v_corrected["dW" + str(l+1)] = v["dW" + str(l+1)] / (1-beta1)
	v_corrected["db" + str(l+1)] = v["db" + str(l+1)] / (1-beta1)
	s["dW" + str(l+1)] = beta2 * s["dW" + str(l+1)] + (1-beta2)* (grads['dW' + str(l+1)] * grads['dW'+str(l+1)])
	s["db" + str(l+1)] = beta2 * s["db" + str(l+1)] + (1-beta2)* (grads['db' + str(l+1)] * grads['db'+str(l+1)])
	s_corrected["dW" + str(l+1)] = s["dW" + str(l+1)] / (1-beta2)
	s_corrected["db" + str(l+1)] = s["dW" + str(l+1)] / (1-beta2)
	parameters["W" + str(l+1)] = parameters["W" + str(l+1)] - learning_rate * v_corrected["dW"+str(l+1)] / (epsilon + np.sqrt(s_corrected["dW" + str(l+1)]))
	parameters["b" + str(l+1)] = parameters["b" + str(l+1)] - learning_rate * v_corrected["db"+str(l+1)] / (epsilon + np.sqrt(s_corrected["db" + str(l+1)]))



## Heroes of Deep Learning  
这一次终于轮到我们中国人接受采访了，接受采访的大佬是林元庆。按照惯例，第一个环节是林元庆大佬介绍他的深度学习经历，在采访中他提到的一点我深有感触，他本科是光学专业，博士选择了转专业到机器学习。他说在那段时光，他每天都在学习新知识，这是非常令人兴奋的，这就和我现在的感觉是一样的，所以我能体会到他的那种被一大堆自己感兴趣的新知识包围时的兴奋和幸福。他现在在国家工程实验室里工作，所做的事情是将一些代码整合到一个平台上，以供学者或者工程上调用，从而减少重复性的成果，从而起到帮助别人研究和工作的作用。整篇采访的时长不长，他讲的大部分都是在工业界如何将深度学习更好的应用，以及如何将这个领域发展的更好、建立一个良性发展循环的过程，而这些都是比较大的概念。最后是对于初学者的概念，林元庆建议从开源的框架开始，我发现这很有趣，在之前采访一些业界学者时，他们都建议从最初的理论和数学公式开始推导，但是在林元庆这种业界大佬来看，他们建议从开源的框架开始，也就是从项目做起。我不知道何者是正确的，我是从理论开始学习的，这就导致学了两个多月的机器学习但是还没有接触到真正的开源框架，但是反过来讲，如果要想真正从事深度的开发或者研究的话，我更认为从理论开始学习，后来再接触已有的框架更好一点。  


---
本博客支持disqus实时评论功能，如有错误或者建议，欢迎在下方评论区提出，共同探讨。  

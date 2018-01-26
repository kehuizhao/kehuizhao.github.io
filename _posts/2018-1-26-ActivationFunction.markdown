---
layout:     post
title:      "深度学习学习笔记——激活函数"
subtitle:   "多种激活函数的对比"
date:       2018-1-26 23:00:00
author:     "ThdLee"
header-img: "img/ActivationFunction/post-bg-DeepLearning.jpg"
catalog:    true
tags:
    - 深度学习
---

# 简介

激活函数（activation function）的引入是为了增强整个网络的表达能力（即非线性）。在神经网络中，每个操作层都是线性的，所以若干层的堆叠只能起到线性映射的作用，无法形成复杂的函数。所以我们在每层叠加完之后，增加一个激活函数，那么输出就变成了一个非线性函数。经过多层的堆叠后，网络就会形成一个复杂的函数，这样网络的表达能力就大大增强了。

由于神经网络中通过感知器来模仿神经元，激活函数实际上是模拟了生物神经元的信号传输特性。在神经科学中，生物神经元通常有一个阈值，当神经元锁获得的输入信号累积效果超出了该阈值，神经元就被激活，处于兴奋状态，否则处于抑制状态。

下面我就介绍几种常见的激活函数。

![activation_function](http://thdlee.com/img/ActivationFunction/activation_function.jpg)

# sigmoid

![sigmoid](http://thdlee.com/img/ActivationFunction/sigmoid.png)

sigmoid非线性函数是曾经较为流行的一种激活函数，它将输入值挤压的0到1范围内，也就是说，当输入一个非常大的正数时，输出接近于1，输入一个非常小的负数时，输出接近于0。因此在使用sigmoid时便有一个严重的问题，即梯度的**饱和效应**（saturation effect）。也就是说，当神经元的激活在接近0或1时会产生饱和，即在这些区域，梯度几乎为0。我们知道，在网络反向传播的过程中，这个局部梯度将会与返回的上游梯度相乘，因此如果局部梯度过小，就会使结果接近零，从而不能使反向传播继续进行，进而导致网络无法训练。此外，在初始化权重时还要特别留意，比如权重过大可能直接引发梯度饱和效应而使网络无法训练。

另外，sigmoid是一个非零中心函数。![non-centered](http://thdlee.com/img/ActivationFunction/non-centered.png)如果x全部为正，这就相当于把上游梯度的符号传了回来，那么w的梯度要么全为正，要么全为负。这将会导致梯度下降时，权重更新会成z字型的下降。不过整个批量的数据的梯度叠加起来后对于权重最终的更新会朝着最佳的权重更新，所以该问题并不怎么严重，只是效率比较低。我们一般使用均值为0的数据，这样数据中既有正值又有负值，这样就不会出现前面的问题。

除此之外，sigmoid函数中使用指数函数，所以计算代价较大。

# tanh

![tanh](http://thdlee.com/img/ActivationFunction/tanh.png)

它将输入值压缩到[-1,1]之间，和sigmoid一样，它也存在梯度饱和问题，会出现梯度消失的现象，不过tanh函数是以0为中心的。所以它比sigmoid函数好一些，但是还是存在一些问题。



# ReLU

![ReLU](http://thdlee.com/img/ActivationFunction/ReLU.png)

ReLU是我们常用的一种激活函数，将它与sigmoid和tanh函数进行对比，会发现当输入为正时，函数不会出现饱和现象，这是它的一个优势。另外它的运算比较简单，sigmoid与tanh中存在指数，所以运算复杂，而ReLU只是简单的max操作，在速度上会非常快。所以使用ReLU在收敛速度上会比sigmoid和tanh快。另外也有实验说明ReLU更具有生物学上的合理性。

它的缺点也很明显，不是以零为中心的函数，tanh解决的问题又出现了。另外，它在正半轴上虽然解决了饱和问题，但是在负半轴上依然存在梯度消失的问题（一般x为0时，梯度为0）。而且在训练的时候，ReLU单元比较脆弱，会产生dead ReLU，在此状态下神经元无法被激活和更新。例如，权重的初始化和学习率过大时都会产生这种现象。

# Leaky ReLU

![Leaky_ReLU](http://thdlee.com/img/ActivationFunction/Leaky_ReLU.png)

Leaky ReLU是为了解决dead ReLU问题而对ReLU的改进。它和ReLU非常相似，唯一不同的地方在于它给出了一个很小的负数梯度值，这也就解决了之前的问题，没有了饱和现象，而且计算任然非常高效，收敛速度也很快。但一些研究者的论文也指出这个激活函数不是很稳定。

---

虽然激活函数多种多样，但是一般情况下使用ReLU函数就可以了，如果出现了dead ReLU的现象，那么可以尝试Leaky ReLU已经其他ReLU的变体来替代ReLU。值得注意的是，在同一个网络中混用不同的激活函数非常少见。
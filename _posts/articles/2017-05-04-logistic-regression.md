---
layout: post
title: 逻辑回归模型 
excerpt: "逻辑回归模型、优化目标和牛顿迭代法"
modified: 2017-07-04
categories: articles
tags: [machine-learning]
comments: true
share: true
---
# 逻辑回归(logistic regression)

## 模型

逻辑回归的模型为
\\[
h_\theta(x) = g(\theta^Tx) = \frac{1}{1+e^{-\theta^Tx}} \tag{1}
\\]
其中 
\\[
g(z)=\frac{1}{1+e^{-z}}
\\]

\\(g(z)\\) 是 \\(\mathbb{R}\\) 上的连续函数，图象类似一个 S 型，因而被称为 S 型函数（Sigmoid function）。

## 优化目标

假设

$$
\begin{align}
P(y=1 \mid x; \theta) &= h_\theta(x) \\\\
P(y=0 \mid x; \theta) &= 1- h_\theta(x)
\end{align}
$$

即
\\[
P(y \mid x; \theta) = (h_\theta(x))^y(1-h_\theta(x))^{(1-y)}
\\]
其取对数后的似然函数为

$$
\begin{align}
\ell(\theta) &= \log L(\theta) \\\\
&= \log \prod_{i=1}^{m}p(y^{(i)} \mid x^{(i)}; \theta) \\\\
&= \sum_{i=1}^{m}y^{i}\log h(x^{(i)})+(1-y^{(i)}) \log (1-h(x^{(i)}))  \tag{2}
\end{align}
$$

优化目标是选择合适的 \\(\theta\\)，使 \\(\ell(\theta)\\) 最大。

## 牛顿迭代法（Newton's Method）

除了线性回归中使用的梯度下降法之外，也可以使用牛顿迭代法。

举例来说，为了找到 \\(\theta \in \mathbb{R}\\)，使得 \\(f(\theta)=0\\)，可以按照以下算法迭代直至 \\(\theta\\) 收敛。
\\[
\theta := \theta - \frac{f(\theta)}{f'(\theta)}
\\]
为了得到 \\(\ell(\theta)\\) 的最大值，应该找到 \\(\ell ’(\theta)=0\\) 的一点（\\(\ell(\theta)\\) 是凸函数）。

牛顿迭代法可以推广到 \\(\theta\\) 是向量时的情形
\\[
\theta := \theta - H^{-1}\nabla_\theta\ell(\theta)
\\]
其中 \\(H\in\mathbb{R}^{n\times n}\\) 称为 **Hessian Matrix**，有
\\[
H_{ij} = \frac{\partial^2\ell(\theta)}{\partial\theta_i\partial\theta_j}
\\]
相比梯度下降法，牛顿迭代法可以更快地收敛，不过计算一个 \\(n\\) 阶矩阵并求其逆矩阵也是不小的计算量。当用牛顿迭代法求对数似然函数的最大值时，该算法也被称为 **Fisher Scoring**。

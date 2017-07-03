---
layout: post
title: 线性回归模型 
excerpt: "Just about everything you'll need to style in the theme: headings, paragraphs, blockquotes, tables, code blocks, and more."
modified: 2017-07-03T14:17:25-04:00
categories: articles
tags: [machine-learning]
comments: true
share: true
---
# 线性回归（Linear Regression）
## 模型与优化目标
线性回归有假设
$$
h_\theta(x) = \sum_{i=0}^{n}\theta_ix_i = \theta^Tx \tag{1}
$$
其中 $n$ 为属性（feature）个数，$\theta, x \in \mathbb{R}^{n+1}$($x_0=1$，相应地 $\theta_0$ 为常数项系数)。

其损失函数（cost function）为
$$
J(\theta) = \frac{1}{2}\sum_{i=1}^{m}(h_\theta(x^{(i)})-y^{(i)})^2 \tag{2}
$$
$m$ 为样本个数。

优化目标是求得参数向量 $\theta$，使损失函数最小。

## 梯度下降法（Gradient Descent）
为了最小化 $J(\theta)$，可以使用梯度下降法。梯度下降法从初始向量 $\theta$ 开始，不断迭代，直到 $J(\theta)$ 收敛。迭代过程中，按照如下算法更新 $\theta$：
$$
\theta_j := \theta_j-\alpha\frac{\partial}{\partial \theta_j}J(\theta) 
$$
其中，$\alpha$ 为学习率（learning rate）。上式也可以写成向量形式
$$
\theta := \theta - \alpha \nabla_\theta J 
$$

## 标准方程（Normal Equation）

用矩阵形式表示 $J(\theta)$ 
$$
J(\theta) = \frac{1}{2}(X\theta - \boldsymbol{y})(X\theta - \boldsymbol{y})^T
$$
其梯度可表示为
$$
\nabla_\theta J(\theta) = X^TX\theta - X^T\boldsymbol{y}
$$
令梯度等于 $\boldsymbol{0}$ 向量，得
$$
\theta = (X^TX)^{-1}X^T\boldsymbol{y}
$$

## 为什么取这样的损失函数？

用如下方程表示 $x^{(i)}, y^{(i)},\theta$ 之间的关系


$$
y^{(i)} = \theta^T x^{(i)} + \epsilon^{(i)}
$$
其中 $\epsilon^{(i)}$ 为误差项（error term）。假设各 $\epsilon^{(i)}$ 之间相互独立，且服从同样的正态分布（即独立同分布， independently and identically distributed，IID），$\epsilon^{(i)} \sim \mathcal{N}(\mu, \sigma^2)$，那么在给定了 $x^{(i)}, \theta$ 条件下的 $y^{(i)}$ 也服从正态分布，即 $y^{(i)} \mid x^{()}; \theta \sim \mathcal{N}(\theta^Tx^{(i)},\sigma^2)$ 。

在给定了 $X, \theta$ 的条件下，取到向量 $\boldsymbol{y}$ 的概率是多少呢？由于其相互独立，可以用以下似然函数（likelihood function）来表示
$$
\begin{align}
L(\theta) &= L(\theta; X, \boldsymbol{y})=p(\boldsymbol{y} \mid X;\theta) \\ 
&=\prod_{i=1}^{m}p(y^{(i)} \mid x^{(i)};\theta) \\ 
&=\prod_{i=1}^{m} \frac{1}{\sqrt{2\pi}\sigma}\text{exp}(-\frac{(y^{(i)}-\theta^Tx^{(i)})^2}{2\sigma^2})
\end{align}
$$
参数优化的目标，就是取得合适的 $\theta$，使得似然函数值最大。由于连乘不便于计算，我们对以上似然函数取自然对数
$$
\begin{align}
\ell(\theta) &= \ln L(\theta) \\
&= m\ln\frac{1}{\sqrt{2\pi}\sigma} - \frac{1}{\sigma^2}\cdot\frac{1}{2}\sum_{i=1}^{m}(y^{(i)}-\theta^Tx^{(i)})^2 \\
&= m\ln\frac{1}{\sqrt{2\pi}\sigma} - \frac{1}{\sigma^2}\cdot J(\theta)
\end{align}
$$
为了使 $\ell(\theta)$ 最大，$J(\theta)$ 必须取最小值，而这里的 $J(\theta)$ 正是 $\text{(2)}$ 式中给出的损失函数。

# 逻辑回归

## 模型

逻辑回归的模型为
$$
h_\theta(x) = g(\theta^Tx) = \frac{1}{1+e^{-\theta^Tx}} \tag{1}
$$
其中 
$$
g(z)=\frac{1}{1+e^{-z}}
$$

$g(z)$ 是 $\mathbb{R}$ 上的连续函数，图象类似一个 S 型，因而被称为 S 型函数（Sigmoid function）。

## 优化目标

假设
$$
\begin{align}
P(y=1 \mid x; \theta) &= h_\theta(x) \\\\
P(y=0 \mid x; \theta) &= 1- h_\theta(x)
\end{align}
$$
即
$$
P(y \mid x; \theta) = (h_\theta(x))^y(1-h_\theta(x))^{(1-y)}
$$
其取对数后的似然函数为
$$
\begin{align}
\ell(\theta) &= \log L(\theta) \\\\
&= \log \prod_{i=1}^{m}p(y^{(i)} \mid x^{(i)}; \theta) \\\\
&= \sum_{i=1}^{m}y^{i}\log h(x^{(i)})+(1-y^{(i)}) \log (1-h(x^{(i)}))  \tag{2}
\end{align}
$$
优化目标是选择合适的 $\theta$，使 $\ell(\theta)$ 最大。

## 牛顿迭代法（Newton's Method）

除了线性回归中使用的梯度下降法之外，也可以使用牛顿迭代法。

举例来说，为了找到 $\theta \in \mathbb{R}$，使得 $f(\theta)=0$，可以按照以下算法迭代直至 $\theta$ 收敛。
$$
\theta := \theta - \frac{f(\theta)}{f'(\theta)}
$$
为了得到 $\ell(\theta)$ 的最大值，应该找到 $\ell ’(\theta)=0$ 的一点（$\ell(\theta)$ 是凸函数）。

牛顿迭代法可以推广到 $\theta$ 是向量时的情形
$$
\theta := \theta - H^{-1}\nabla_\theta\ell(\theta)
$$
其中 $H\in\mathbb{R}^{n\times n}$ 称为 **Hessian Matrix**，有
$$
H_{ij} = \frac{\partial^2\ell(\theta)}{\partial\theta_i\partial\theta_j}
$$
相比梯度下降法，牛顿迭代法可以更快地收敛，不过计算一个 $n$ 阶矩阵并求其逆矩阵也是不小的计算量。当用牛顿迭代法求对数似然函数的最大值时，该算法也被称为 **Fisher Scoring**。

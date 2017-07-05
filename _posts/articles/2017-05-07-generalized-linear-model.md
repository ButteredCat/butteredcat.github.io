---
layout: post
title: 广义线性模型 
excerpt: "从广义线性模型中，可以自然地推导出线性回归和逻辑回归等特殊情形下的模型。"
modified: 2017-07-05
categories: articles
tags: [machine-learning]
comments: true
share: true
---
# 广义线性模型（Generalized Linear Model）

前面讲到的用于回归的线性回归和用于分类的逻辑回归算法模型，是怎么得到的呢？尤其是那个 Sigmoid 函数，总不该是拍脑袋想出来的。其实，他们都是广义线性模型（Generalized Linear Model）的特殊形式。

## 指数族分布（The exponential family）

不管是线性回归中涉及的正态分布，还是逻辑回归中的伯努利分布，都属于指数族分布。其具有如下形式：
\\[
p(y; \eta) = b(y)\exp(\eta^TT(y)-a(\eta)) \tag{1}
\\]
其中，\\(\eta\\) 称为**自然参数**（**natural parameter**，或**canonical parameter**），\\(T(y)\\) 是 \\(y\\) 的**充分统计量**（**sufficient statistic**）。

举例来说，在逻辑回归中，\\(y\\) 服从参数为 \\(\phi\\) 的伯努利分布，也叫 \\((0-1)\\) 分布，即
\\[
p(y=1; \phi) = \phi \\\\
p(y=0; \phi) = 1 - \phi
\\]
经过变换，可以改写成指数族分布的形式

$$
\begin{align}
p(y; \phi) &= \phi^{y}(1-\phi)^{1-y} \\\\
&= \exp\left(y\ln\left(\frac{\phi}{1-\phi}\right)+\ln(1-\phi)\right)
\end{align}
$$

对比 \\((1)\\) 式，可知

$$
\begin{align}
\phi &= 1/(1+e^{-\eta}) \tag{2} \\\\
T(y) &=y \\\\
a(\eta) &= \ln(1+e^{\eta}) \\\\
b(y) &= 1
\end{align}
$$

## 构造 GLM

广义线性模型有3个基本假设：

1. \\(y \mid x;\theta\\) 服从自然参数为 \\(\eta\\) 的指数族分布。
2. 模型的预测目标是在给定 \\(x​\\) 的条件下，输出 \\(y​\\) 的充分统计量 \\(T(Y)​\\) 的数学期望 \\(E[T(y)\mid x]​\\)。由于在很多条件下 \\(T(y)=y​\\)，我们的目标可以认为是 \\(h(x) = E[y\mid x]​\\)。
3. \\(\eta = \theta^Tx\\)


还是以逻辑回归为例，由上述假设，并结合式 \\((2)\\)，可知

$$
\begin{align}
h_\theta(x) &= E[y \mid x; \theta] \\\\
&= \phi \\\\
&= 1/(1+e^{-\eta}) \\\\
&= 1/(1+ e^{-\theta^T x})
\end{align}
$$

得到的结果正是我们之前使用的逻辑回归模型。

----

广义线性模型的意义是，已知 \\(y \mid x\\) 的分布（正态、伯努利、泊松等等），可以由假设自然地推出模型，之后再写出似然函数，最大化似然函数估计模型参数。以上提到的线性回归损失函数为什么是平方和而不是绝对值和、四次方和的形式，逻辑回归模型为什么是那样一种形式，都在广义线性模型中有了合理的数学解释。


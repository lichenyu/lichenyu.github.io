title: 最大似然与逻辑回归
date: 2019/10/24
categories:
- Machine Learning
tags:
- Model
---


概率和似然是相反的过程：
- 概率是已知参数或条件，预测可能发生的结果（推理）；
- 似然是已知（多条）观测到的结果，估计相关的参数或条件（训练）。

最大似然估计，即选择一组参数，使得观测到的结果对应最大出现概率。

以离散型分布为例，$\theta$是带估计参数，$p(o; \theta)$是观测结果为$o$的概率，对于一系列观测到的结果$o_1, o_2, o_3, ..., o_n$，联合分布概率为
$$
L(\theta) = \prod_{i=1}^{n}p(o_i; \theta)
$$
将这个联合分布概率$L(\theta)$称为**似然函数**。
最大似然估计即为求$\theta$值，对应联合分布概率最大，即观测结果是最可能出现的场景。
对似然函数取对数
$$
\ln L(\theta) = \ln L(\theta_1, \theta_2, \theta_3, ..., \theta_k) = \sum_{i=1}^{n} \ln p(o_i; \theta_1, \theta_2, \theta_3, ..., \theta_k)
$$
最大化此式即可（求导数、令导数为0得到似然方程、解似然方程得到参数，或使用梯度下降法等）。

---

对于逻辑回归
$$
\begin{align*}
&h_\theta(x) = \frac{1}{1+e^{-\theta^Tx}} \\
&P(y=1|x;\theta) = h_\theta(x) \\
&P(y=0|x;\theta) = 1-h_\theta(x)
\end{align*}
$$
其中，$\theta$是一些列参数，$x$是特征，$y$是标签，$y=1|x$、$y=0|x$为某一次观测结果（在特征为x时，标签为y）
将概率写成一般形式（单次观测）：
$$
P(y|x;\theta) = (h_\theta(x))^y(1-h_\theta(x))^{1-y}
$$
似然函数则为：
$$
\begin{align*}
L(\theta) &= \prod_{i=1}^{n}p(y_i|x_i; \theta) \\
& = \prod_{i=1}^{n}(h_\theta(x_i))^{y_i}(1-h_\theta(x_i))^{1-y_i}
\end{align*}
$$
求最大似然，取log：
$$
\ln L(\theta) = \sum_{i=1}^{n} {y_i} \ln (h_\theta(x_i)) + (1-y_i) \ln (1-h_\theta(x_i))
$$
最大化此式即可。

多说一句，为什么逻辑回归求参数方法，使用最大似然法，而不是最小二乘法？
如果用最小二乘法，目标函数就是
$$
\sum_{i=1}^{n}(y_i - \frac{1}{1+e^{-\theta^Tx_i}})^2
$$
是非凸的，不容易求解，会得到局部最优。
如果用最大似然估计，目标函数就是对数似然函数，是关于$\theta$的高阶连续可导凸函数，可以方便通过一些凸优化算法求解，比如梯度下降法、牛顿法等。

title: ARMA ARIMA
date: 2019/10/11
categories:
- Data Analysis
tags:
- Algorithm
---


## 平稳

AR/MA/ARMA用于分析平稳时间序列，ARIMA通过差分可以用于处理非平稳时间序列。

平稳的直观理解：
平稳时间序列围绕一个常数上下波动的；
而一般具有长期趋势的时间序列都是非平稳时间序列。根据趋势的不同，可以使用差分将具有长期趋势的时间序列转换成平稳时间序列。例如，时间序列的数值为线性增长的$(1,2,3,4,5,6,7,8)$，经过一阶差分以后，新的时间序列的数值为$(1,1,1,1,1,1,1)$，就成为稳定的时间序列了。

## ARMA

### AR (AutoRegressive)

$$
X_t = c + \sum_{i=1}^{p}\varphi_i X_{t-i} + \varepsilon_t
$$

自回归模型描述的是当前值与历史值之间的关系。
利用历史值与当前值的相关关系（自相关），用之前的$p$个历史值的线性组合（$p$阶）来预测当前值。
$c$为常数项。
$\varepsilon_t$为当前时刻的白噪声干扰项。白噪声可以理解为时间序列值的随机波动，这些随机波动的总和会等于0。

AR模型思想很简单，该模型认为通过时间序列过历史时刻点的线性组合加上白噪声即可预测当前时刻点。

### MA (Moving Average)

$$
X_t = \mu + \varepsilon_t + \sum_{i=1}^{q}\theta_i \varepsilon_{t-i}
$$

$\mu$是$X_t$的均值（often assumed to equal 0）, $\varepsilon _{t}, \varepsilon _{t-1}, ...$是白噪声。
MA模型和AR大同小异，它并非是历史时刻点的线性组合，而是历史白噪声的线性组合（q阶）。与AR最大的不同之处在于，AR模型中历史白噪声的影响是间接影响当前预测值的（通过影响历史时序值）。

移动平均过程其实可以作为自回归过程的补充，解决自回归方程中白噪声的求解问题，两者的组合就成为自回归移动平均过程，称为ARMA模型。

### ARMA

将AR和MA模型混合可得到ARMA模型，AR(p)和MA(q)共同组成了ARMA(p,q)。

$$
X_t = c + \varepsilon_t + \sum_{i=1}^{p}\varphi_i X_{t-i} + \sum_{i=1}^{q}\theta_i \varepsilon_{t-i}
$$

$c$为常数项。
$\varepsilon _{t}, \varepsilon _{t-1}, ...$是白噪声。
AR可以解决当前数据与后期数据之间的关系，MA则可以解决随机变动也就是噪声的问题（不只是白噪声，噪声由白噪声MA来估算）。

那么在建模过程中应该如何选择ARMA模型的最佳超参数p和q呢？
最常用的技术是采用循环在p和q各自0到5（或者更大）的范围内搜索最小AIC或BIC的(p,q)组合。

（忽略常数项$c$），滞后算子表示法：

$$
(1 - \sum_{i=1}^{p}\varphi_i L^i)X_t = (1 + \sum_{i=1}^{q}\theta_i L^i)\varepsilon_t
$$

其中$L$是滞后算子（Lag operator），$L^iX_t = X_{t-i}$。


## ARIMA

AR/MA/ARMA模型适用于平稳时间序列的分析，当时间序列存在上升或下降趋势时，这些模型的分析效果就大打折扣了，这时差分自回归移动平均模型也就应运而生。
ARIMA模型能够用于齐次非平稳时间序列的分析，这里的齐次指的是原本不平稳的时间序列经过d次差分后成为平稳时间序列。

ARIMA(p,d,q)，在d阶差分后进行ARMA：

$$
(1 - \sum_{i=1}^{p}\varphi_i L^i)(1 - L)^d X_t = (1 + \sum_{i=1}^{q}\theta_i L^i)\varepsilon_t
$$

在ARIMA模型的基础上可以衍生出SARIMA模型，SRIMA模型能够刻画季节效应，如商品价格的周期性变动。



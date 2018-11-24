title: KPI Check Note
date: 2018/09/30
categories:
- Data Analysis
tags:
- KPI Check
---


## 依据 ##

- 当前曲线变化趋势与baseline应该相似
  - 周期性
  - 同质性
- 当前曲线变化趋势应该较平缓
  - 平滑性

---

## 数据预处理 ##

### [线性插值] ###

### [周期性判断] ###

利用自相关函数来进行判断


---

## 毛刺检测 ##

### [数据自身 + k-sigma准则] ###

### [对比滤波 + k-sigma准则] ###

*曲线可以小幅度震荡。（高频低通）滤波后曲线与真实曲线的差值（绝对值），在k-sigma以外的点为异常。*

**滤波：**

- 中值滤波 Median Filter
  - 某点的值为该点周围一个窗口的中值，迭代滑动窗口。
- 滑动平均 Moving Average
  - 某点的值为该点前面一个窗口的平均值，迭代滑动窗口。
- 指数平滑 Exponential Smoothing
  - 一次指数平滑：适用于趋势无明显变化的时间序列
{% math %}
\begin{aligned}
s_t &= \alpha x_{t} + (1-\alpha)s_{t-1}\\
&= \alpha x_{t} + \alpha (1-\alpha)x_{t-1} + (1 - \alpha)^2 s_{t-2}\\
&= \alpha \left[x_{t} + (1-\alpha)x_{t-1} + (1-\alpha)^2 x_{t-2} + (1-\alpha)^3 x_{t-3} + \cdots + (1-\alpha)^{t-1} x_{1} \right] + (1-\alpha)^{t} x_0
\end{aligned}
{% endmath %}
  - 二次指数平滑：适用于线性趋势的时间序列
- 卡尔曼滤波 Kalman Filter

**k-sigma准则，通过统计理论检测异常点：**

- $3-\sigma$：在正态分布中，数值分布在$(\mu - 3\sigma, \mu + 3\sigma)$中的概率约为0.9974。$\mu$为均值、$\sigma$为标准差。

---

## 周期性数据横向对比 ##

### [对比同时刻点 + k-sigma准则] ###

*不同周期中同时刻的点相差不大。对同时刻各点，通过k-sigma准则确定阈值，过滤异常。*


### [FFT提取频域特征 + KS检验] ###

同一时刻各点，以及之前若干点，组成时间窗。

*不同周期中，同时刻时间窗的点，频域的统计分布应类似。对同时刻各时间窗做FFT变换，统计分布，不符合同一分布则为异常。*

**快速傅里叶变换 FFT:**

- 时域转频域。

**KS检验 Kolmogorov-Smirnov Test:**

- 基于CDF检验一个观测是否符合某一分布，或两个观测是否符合同一分布。

---

## 参数选择 ##

- 网格搜索 Grid Search
  - 遍历
- 模拟退火 Simulated Annealing
  - 若移动后得到更优解，则总是接受该移动；若移动后的解比当前解要差，则以一定的概率接受移动，而这个概率随着时间（迭代次数）推移而逐渐降低。初始时设置一个初始温度T和降温速度r，降温到一定程度则停止迭代。
  - 具体实现时，一定概率接受体现为：比较random(0,1)与计算出来的（接受移动）概率值。
  - 对比贪心策略：若移动后得到更优解，则总是接受该移动；否则，不移动。模拟退火能够一定程度上避免局部最优解困局。

---

## 异常点检测方法 ##

### [统计] ###

- k-sigma准则选取阈值，[使用场景shape=(时刻, 单指标)]
  - $\mu$为均值、$\sigma$为标准差。
  - k-sigma准则：在$(\mu - k \times \sigma, \mu + k \times \sigma)$范围外的数值为outlier。
  - $k = 3$：在正态分布中，数值分布在$(\mu - 3\sigma, \mu + 3\sigma)$中的概率约为0.9974。
- Tukey Fences选取阈值，[使用场景shape=(时刻, 单指标)]
  - When there are no outliers in a sample, the mean and standard deviation are used to summarize a typical value and the variability in the sample, respectively.  When there are outliers in a sample, the median and interquartile range are used to summarize a typical value and the variability in the sample, respectively.
  - Interquartile Range $ IQR = Q3 - Q1$
  - Tukey Fences: Outliers are values below $Q1 - k \times IQR$ or above $Q3 + k \times IQR$, where $k = 1.5$ indicates an "outlier", and $k = 3$ indicates data that is "far out".
  - Q3 is positioned at $0.675\sigma$ for a normal distribution. The $IQR$ represents $2 \times 0.675\sigma = 1.35\sigma$. The outlier fence is determined by adding $Q3$ to $1.5 \times IQR = 0.675\sigma + 1.5 \times 1.35\sigma = 2.7\sigma$. This level would declare 0.7% of the measurements to be outliers.
- 基于$\chi^{2}$检验的分布相似度检测，[使用场景shape=(时刻, 多指标)]
  - 检测各指标出现频次的分布，相较于（训练）基线数据，是否产生了变动。如果某时刻的指标分布与基线不符，则该时刻异常。
  - {% post_link chi2_test 代码示例 %}给出了计算各个指标的卡方值，将卡方值较大的指标判定为异常的指标。此外，也可以累加了各个指标的卡方值，这样不关注哪个指标是异常的，而是关注哪个（按行累加的行）时刻是异常的。
- 基于KS检验的频域分布相似度检测，[使用场景shape=(时刻, 单指标)]
  - 加窗FFT变换，获取（一段）序列的频域信息。
  - KS检验，基于CDF检验两序列是否来自同一分布。

### [聚类] ###

- k-means/x-means，通过聚类检测异常点，[常用场景shape=(时刻, 多指标)]
- DBSCAN，通过聚类检测异常点，[常用场景shape=(时刻, 多指标)]
  - 检测“邻域半径内的点个数”（即密度）。对满足密度要求的核心点，其邻域半径内所有点为新核心点，递归拓展“邻域-核心点”。

### [离群点识别] ###

- 局部异常因子 Local Outlier Factor, LOF，[常用场景shape=(时刻, 单指标)或(时刻, 多指标)]
  - {% post_link local_outlier_factor 算法介绍 %}
  - 在LOF之前的异常检测算法大多是基于统计方法、或借用聚类算法。但是，基于统计的异常检测算法通常需要假设数据服从特定的概率分布，这个假设往往是不成立的。而聚类的方法通常只能给出0/1判断（即是否为异常点），不能量化每个数据点的异常程度。相比较而言，基于密度的LOF算法要更简单、直观。它不需要对数据的分布做太多要求，还能量化每个数据点的异常程度。
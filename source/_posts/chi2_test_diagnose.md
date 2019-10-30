title: 卡方值检验两序列是否来自同一分布
date: 2018/10/29
categories:
- Data Analysis
tags:
- Algorithm
---


卡方检验常用于检查某序列是否属于某分布。
计算（整体）卡方值，如果卡方值较大，则认为不属于同一分布。
{% math %}
\begin{aligned}
\chi^{2} = \sum \frac{(A-E)^{2}}{E} = \sum_{i=1}^{k}\frac{(A_{i}-E_{i})^{2}}{E_{i}}
\end{aligned}
{% endmath %}
其中，$A_{i}$为各点（指标）出现频次，$E_{i}$为期望。

检验两序列是否来自同一分布的常用场景为：一个序列为训练集累加获取的基线序列（因为是对分布情况的分析，累加即可无需平均）；而另一个序列为待检验的测试序列。
期望使用两序列频次求和后平均来计算。
对各点求算两序列的卡方值（之和），较大的点为异常指标点。
也可直接对序列的所有点卡方值累加，比较各点卡方值之和，如果检测序列的各点sum值较大，则认为检测序列整体上分布于训练模板不同。

```python
# -*- coding: UTF-8 -*-
import numpy as np
import pandas as pd


def calc_chi2_value(data):
    S = data.values.sum()
    # SA = data.iloc[:, 0].sum()
    # SB = data.iloc[:, 1].sum()
    # A、B出现错误码总数
    SA, SB = data.sum()

    # A、B出现错误码总数比例
    PA = SA / (S * 1.0)
    PB = SB / (S * 1.0)

    # 各错误码出现总数list
    Si = data.sum(axis=1)

    # A、B各错误码出现总数预期
    EAi = Si * PA
    EBi = Si * PB

    E = np.hstack((EAi.values.reshape(EAi.shape[0], 1), EBi.values.reshape(EBi.shape[0], 1)))

    # 对A、B各点计算卡方值
    chis = (data - E) * (data - E) / E

    # 各个错误码
    CHIi = np.nansum(chis, axis=1)

    chi_values = pd.DataFrame(CHIi, columns=["value"], index=data.index)

    return chi_values


class Chi2:

    def __init__(self, topk=10):
        self.topk = topk
        self.base_value = None

    """
    the base matrix.
  
    data: one column dataframe
    """
    def train(self, data):
        self.base_value = data

    """
    the data matrix
  
    data: one column dataframe
    return the chi2 values for each sample.
    """
    def predict(self, data):
        ch_matrix = pd.concat([self.base_value, data], axis=1)
        return calc_chi2_value(ch_matrix)

    """
      the data matrix
  
      data: one column dataframe
      return the chi2 values for each sample and the index of the topk
      """
    def predict_topk(self, data):
        chi_values = self.predict(data)
        ranks = chi_values['value'].rank(ascending=False)
        topk_index = ranks <= self.topk
        return chi_values, topk_index


if __name__ == "__main__":
    df = pd.DataFrame(np.arange(20).reshape((10, 2)))

    chi2 = Chi2(2)
    chi2.train(pd.DataFrame([1, 2, 3, 4, 5]))

    chi_values, ranks = chi2.predict_topk(pd.DataFrame([12, 4, 6, 8, 10]))
    print(chi_values)
    print(ranks)
```
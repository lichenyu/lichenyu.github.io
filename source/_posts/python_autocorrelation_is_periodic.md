title: Python - 基于自相关函数的周期性检测方法
date: 2019/04/11
categories:
- Program Development
tags:
- Python
---


## 核心思想 ##

求自相关函数，比较第0个值（无延迟）与可能周期位置（或第一个峰值）的值是否相近

```python
from statsmodels.tsa.stattools import acf

rx = acf(x,fft=True, unbiased=True, nlags=np.size(x)-1)

if param['if_judge_periodicity_for_known_period'] == True:
	peak_value = rx[param['period_size']]
else:
	peaks_detected = peakdetect(rx)[0]	# 预估周期大概有多
	peak_value = peaks_detected[0][1]	# 第1个最大峰值的实际值

periodicity_ratio = peak_value/rx[0]

# 后续比较periodicity_ratio与预定义阈值
```
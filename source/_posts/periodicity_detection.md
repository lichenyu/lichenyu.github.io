title: 序列周期性检测
date: 2019/04/11
categories:
- Data Analysis
tags:
- KPI Check
---


## 基于FFT ##

信号的连续/离散性 v.s. 周期性/非周期性，在时域与频域中是相互决定的。即：时域连续，频域非周期；时域离散，频域周期；时域周期，频域离散；时域非周期，频域连续。

因此，将序列做FFT后，如果是准离散信号（体现为大毛刺），则可认为该序列为近似周期性。

基于FFT的周期性检测方法，具体流程为：（1）对输入信号做FFT；（2）检测FFT结果离散性（是否存在大毛刺）；（3）根据FFT结果是否为离散，给出周期性判断结果。

{% post_link python_fft_is_periodic 代码示例 %}

## 基于自相关函数 ##


{% post_link python_autocorrelation_is_periodic 代码示例 %}

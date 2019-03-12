title: Python - FFT代码示例
date: 2019/03/12
categories:
- Program Development
tags:
- Python
---


## FFT变换、反变换、画图 ##

```python
# -*- coding: UTF-8 -*-

import numpy as np
from scipy.fftpack import fft, ifft
import matplotlib.pyplot as plt

if __name__ == '__main__':

    """
    首先，连续/离散 v.s. 周期性/非周期性，在时域与频域中是相互决定的。
    时域连续，频域非周期性；时域离散，频域周期性；
    时域周期性，频域离散；时域非周期性，频域连续。
    """
    """
    其次，奈奎斯特采样定理：为了不失真地恢复模拟信号，采样频率应该不小于模拟信号频谱中最高频率的2倍。
    （第一个对称中心是采样频率的一半，因为极限场景中，频谱右半部分+频谱左半部分 = 2 * f_max = fs）
    """

    # #----------FFT Demo----------
    # # 采样时长
    # ts = 6.0
    # # 采样点数
    # n = 500
    # # 采样频率
    # fs = ts / n
    # # 时域
    # x = np.linspace(0.0, ts, n, endpoint=False)
    # # 时域信号
    # freq_sampling1 = 10
    # freq_sampling2 = 20
    # amplitude1 = 2
    # amplitude2 = 4
    # y = amplitude1 * np.sin(2 * np.pi * freq_sampling1 * x) + amplitude2 * np.sin(2 * np.pi * freq_sampling2 * x)
    # plt.figure(figsize=(10, 4))
    # plt.plot(x, y, 'k', lw=0.8)
    # plt.show()
    #
    # yf = fft(y)
    # xf = np.linspace(0.0, 1.0 / 2.0 * fs, n // 2, endpoint=False)
    # yf1 = 2.0 / n * np.abs(yf[: n // 2])
    # plt.figure(figsize=(10, 6))
    # plt.plot(xf, yf1)
    # plt.show()
    # exit(0)


    # 采样时长ts = 288s。
    ts = 288.0
    # 采样点数n = 288。
    n = 288
    # 采样频率fs = 1Hz，即1s采1个点。
    fs = n / ts

    # 时域
    x = np.linspace(0.0, ts, n, endpoint=False)
    # 时域信号
    # 周期性噪声
    cycle = [1, 0, 0, 0, 0, 0]
    y1 = np.array(cycle * (n // len(cycle)) + cycle[0 : n % len(cycle)])
    # 混入一些非周期信号
    y2 = np.zeros(n)
    y2[3] = 5
    y2[13] = 8
    y2[34] = 6
    y = y1 + y2
    #y = np.sin(2 * np.pi * 10 * x) + 2 * np.sin(2 * np.pi * 20 * x)
    #y = 1 + 2 * np.cos(2 * np.pi * 50 * x - np.pi * 30 / 180) + 3 * np.cos(2 * np.pi * 100 * x + np.pi * 60 / 180)

    plt.figure(figsize=(15, 6))
    plt.subplot(231)
    plt.plot(x[0:50], y[0:50])
    plt.title('Original Signal (Periodic + Aperiodic)', fontsize=9)
    plt.ylim((-1, 10))
    print("Original Signal")
    print(y)

    yf = fft(y)
    yf1 = fft(y1)
    yf2 = fft(y2)

    xf = np.linspace(0.0, 1.0 * fs, n, endpoint=False)

    # 归一化处理
    yf_n = 2.0 / n * np.abs(yf)
    xf_n = xf
    yf1_n = 2.0 / n * np.abs(yf1)
    xf1_n = xf
    print("FFT Periodic")
    print(yf1_n)
    yf2_n = 2.0 / n * np.abs(yf2)
    xf2_n = xf
    print("FFT Aperiodic")
    print(yf2_n)
    plt.subplot(232)
    plt.plot(xf1_n, yf1_n, 'g')
    plt.title('FFT Transform Normalization (Periodic)', fontsize=9, color='g')
    plt.xlim((-0.01, max(xf_n) + 0.01))
    plt.ylim((0, 1))
    plt.subplot(233)
    plt.plot(xf2_n, yf2_n, 'g')
    plt.title('FFT Transform Normalization (Aperiodic)', fontsize=9, color='g')
    plt.xlim((-0.01, max(xf_n) + 0.01))
    plt.ylim((0, 1))
    plt.subplot(234)
    plt.plot(xf_n, yf_n, 'g')
    plt.title('FFT Transform Normalization (Periodic + Aperiodic)', fontsize=9, color='g')
    plt.xlim((-0.01, max(xf_n) + 0.01))
    plt.ylim((0, 1))

    # 滤除毛刺
    thr = 1. / len(cycle)
    plt.hlines(thr, min(xf_n), max(xf_n), linestyles="dashed")
    idx = np.where(2.0 / n * np.abs(yf) > thr)
    yf.real[idx] = 0
    yf.imag[idx] = 0

    # 归一化处理
    yf_n = 2.0 / n * np.abs(yf)
    xf_n = xf
    plt.subplot(235)
    plt.plot(xf_n, yf_n, 'r')
    plt.title('FFT Transform Normalization (After Filtering)', fontsize=9, color='r')
    plt.xlim((-0.01, max(xf_n) + 0.01))
    plt.ylim((0, 1))

    # 还原时域
    filtered_y = ifft(yf).real
    plt.subplot(236)
    plt.plot(x[0:50], filtered_y[0:50], 'b')
    plt.title('Filtered Signal', fontsize=9, color='b')
    plt.ylim((-1, 10))
    print("Filtered Signal")
    print(filtered_y)

    plt.show()


```

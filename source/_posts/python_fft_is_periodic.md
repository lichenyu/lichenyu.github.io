title: Python - 基于FFT的周期性检测方法
date: 2019/03/13
categories:
- Program Development
tags:
- Python
---


## 核心思想 ##

信号的连续/离散性 v.s. 周期性/非周期性，在时域与频域中是相互决定的。即：时域连续，频域非周期；时域离散，频域周期；时域周期，频域离散；时域非周期，频域连续。

基于FFT的周期性检测方法，具体流程为：（1）对输入信号做FFT；（2）检测FFT结果离散性；（3）根据FFT结果是否为离散，给出周期性判断结果。

```python
# -*- coding: UTF-8 -*-

import numpy as np
from scipy.fftpack import fft, ifft
import matplotlib.pyplot as plt
import time
import random


def is_periodic(arr, z_value, frac_thr):
    starttime = time.time()
    f_arr = fft(arr)
    f_arr = 2.0 / n * np.abs(f_arr)
    idx_z = np.where(f_arr <= z_value)
    print("idx_z count: %d, f_arr len: %d" % (len(idx_z[0]), len(f_arr)))
    endtime = time.time()
    print("time cost = %fs" % (endtime - starttime))
    return len(idx_z[0]) * 1.0 / len(f_arr) >= frac_thr


def is_periodic2(arr, thr=10):
    starttime = time.time()
    f_arr = fft(arr)
    f_arr = 2.0 / n * np.abs(f_arr)
    median = np.median(f_arr)
    max = np.max(f_arr)
    print("median: %f, max: %f" % (median, max))
    endtime = time.time()
    print("time cost = %fs" % (endtime - starttime))
    return max * 1.0 / median >= thr


if __name__ == '__main__':
    # 采样时长ts = 288s。
    ts = 288.0
    # 采样点数n = 288。
    n = 288
    # 采样频率fs = 1Hz，即1s采1个点。
    fs = n / ts

    # 时域
    x = np.linspace(0.0, ts, n, endpoint=False)

    # ---------- 生成一段周期性数据 ----------
    # 周期性冲击
    y1 = np.zeros(n)
    for i in range(0, n, 6):
        y1[i] = random.randint(8, 12) * 1.0 / 10

    # 正弦波 （相比而言，冲击总是正信号，有直流分量，频域0处会有较大值）
    # y1 = np.sin(2 * np.pi * 0.1 * x) + 2 * np.sin(2 * np.pi * 0.2 * x)
    
    # 近似周期性冲击
    # full_list = []
    # T = 7
    # T_count = n // T
    # for _ in range(0, T_count):
    #     cur_list = [0] * random.randint(T - 1, T)
    #     cur_list[0] = random.randint(8, 12) * 1.0 / 10
    #     full_list = full_list + cur_list
    # if len(full_list) > n:
    #     full_list = full_list[: n]
    # else:
    #     full_list = full_list + [0] * (n - len(full_list))
    # y1 = np.array(full_list)

    # ---------- 生成一段非周期性数据 ----------
    y2 = np.zeros(n)
    idx = random.sample(range(0, len(x)), 5)
    for i in idx:
        y2[i] = random.randint(5, 10)

    # ---------- 画图 ----------
    plt.figure(figsize=(15, 6))
    plt.subplot(231)
    plt.plot(x[0:], y1[0:])
    plt.title('Original Signal (Periodic)', fontsize=9)
    plt.ylim((-1, 10))
    plt.subplot(234)
    plt.plot(x[0:], y2[0:])
    plt.title('Original Signal (Aperiodic)', fontsize=9)
    plt.ylim((-1, 10))

    yf1 = fft(y1)
    yf2 = fft(y2)
    xf = np.linspace(0.0, 1.0 / 2.0 * fs, n // 2, endpoint=False)

    # 归一化处理
    yf1_n = 2.0 / n * np.abs(yf1)
    xf1_n = xf
    print("FFT Periodic")
    print(yf1_n)
    yf2_n = 2.0 / n * np.abs(yf2)
    xf2_n = xf
    print("FFT Aperiodic")
    print(yf2_n)

    plt.subplot(232)
    plt.plot(xf1_n, yf1_n[: n // 2], 'g')
    plt.title('FFT Transform Normalization (Periodic)', fontsize=9, color='g')
    plt.ylim((0, 1))
    plt.subplot(235)
    plt.plot(xf2_n, yf2_n[: n // 2], 'g')
    plt.title('FFT Transform Normalization (Aperiodic)', fontsize=9, color='g')
    plt.ylim((0, 1))

    # 周期性检测
    plt.subplot(233)
    # if is_periodic(y1, 0.02, 0.5):
    if is_periodic2(y1):
        plt.title('Periodic Signal Detected!', fontsize=9, color='r')
        plt.plot(xf1_n, yf1_n[: n // 2], 'r')
        print("y1 is periodic!!!".center(100, "-"))
    else:
        plt.title('Aperiodic Signal...', fontsize=9, color='k')
        plt.plot(xf1_n, yf1_n[: n // 2], 'k')
    plt.ylim((0, 1))

    plt.subplot(236)
    # if is_periodic(y2, 0.02, 0.5):
    if is_periodic2(y2):
        plt.plot(xf2_n, yf2_n[: n // 2], 'r')
        plt.title('Periodic Signal Detected!', fontsize=9, color='r')
        print("y2 is periodic!!!".center(100, "-"))
    else:
        plt.title('Aperiodic Signal...', fontsize=9, color='k')
        plt.plot(xf2_n, yf2_n[: n // 2], 'k')
    plt.ylim((0, 1))

    plt.show()


```

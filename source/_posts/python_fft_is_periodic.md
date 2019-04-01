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


def is_periodic(arr, z_value=0.02, frac_thr=0.5):
    starttime = time.time()
    f_arr = fft(arr)
    f_arr = 2.0 / len(arr) * np.abs(f_arr)
    idx_z = np.where(f_arr <= z_value)
    print("idx_z count: %d, f_arr len: %d" % (len(idx_z[0]), len(f_arr)))
    endtime = time.time()
    print("time cost = %fs" % (endtime - starttime))
    return len(idx_z[0]) * 1.0 / len(f_arr) >= frac_thr


def is_periodic2(arr, mmrate_thr=10, show=False, plot=False, filename=""):
    starttime = time.time()

    if type(arr) == list:
        arr = np.array(arr)

    # 准直流
    # if len(np.where(arr > 0)[0]) >= 0.9 * len(arr):
    #    return False

    # 标准化
    arr_mean = np.mean(arr)
    arr_std = np.std(arr)
    if arr_std == 0:
        arr_std = 1
    arr = (arr - arr_mean) / arr_std
    # 二值化
    arr[np.where(arr > 0.01)[0]] = 1
    arr[np.where(abs(arr) <= 0.01)[0]] = 0
    arr[np.where(arr < -0.01)[0]] = -1

    f_arr = fft(arr)
    f_arr = 2.0 / len(arr) * np.abs(f_arr)

    # 去除直流影响，仅考察0.1~0.5
    median = float(np.median(f_arr[len(arr) // 10 : len(arr) // 10 * 5]))
    max = float(np.max(f_arr[len(arr) // 10 : len(arr) // 10 * 5]))
    # 计算max / median的比值，考察频域信号是否为离散冲击
    if median != 0:
        mmrate = max * 1.0 / median
    else:
        if max == 0:
            mmrate = 0
        else:
            mmrate = mmrate_thr + 999
    print("median: %f, max: %f, max_median rate: %f" % (median, max, mmrate))

    endtime = time.time()
    print("time cost = %fs" % (endtime - starttime))

    if show or plot:
        ts = len(arr)
        n = len(arr)
        fs = n * 1.0 / ts
        x = np.linspace(0.0, ts, n, endpoint=False)
        xf = np.linspace(0.0, 1.0 / 2.0 * fs, n // 2, endpoint=False)

        plt.figure(figsize=(15, 6))
        plt.subplot(121)
        plt.plot(x, arr)
        plt.title('Original Signal', fontsize=12)
        #plt.ylim((0, 100))

        plt.subplot(122)
        plt.plot(xf, f_arr[: n // 2], 'g')
        plt.title('FFT Transform Normalization', fontsize=12, color='g')

        if plot:
            plotfile = "%s_%.2f_%.2f_%.2f_" % (mmrate >= mmrate_thr, mmrate, max, median) + filename
            plt.savefig(plotfile)
        if show:
            plt.show()

    return mmrate >= mmrate_thr


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
    for i in range(0, n, 10):
        y1[i] = random.randint(8, 12) * 1.0 / 10

    # 周期性冲击 Type2
    # full_list = []
    # sub_list = [1, 0, 0, 1, 0]
    # T = len(sub_list)
    # T_count = n // T
    # for _ in range(0, T_count):
    #     sub_list[0] = random.randint(8, 12) * 2.0 / 10
    #     sub_list[3] = random.randint(8, 12) * 2.0 / 10
    #     full_list = full_list + sub_list
    # if len(full_list) > n:
    #     full_list = full_list[: n]
    # else:
    #     full_list = full_list + [0] * (n - len(full_list))
    # y1 = np.array(full_list)

    # 正弦波 （相比而言，冲击总是正信号，有直流分量，频域0处会有较大值）
    # y1 = np.sin(2 * np.pi * 0.1 * x) + 2 * np.sin(2 * np.pi * 0.2 * x)

    # 近似周期性冲击 --> 此种信号（周期取值发生震荡）下性能不佳
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
    # 随机冲击
    # y2 = np.zeros(n)
    # idx = random.sample(range(0, len(x)), 5)
    # for i in idx:
    #     y2[i] = random.randint(5, 10)
    # 方波
    y2 = np.zeros(n)
    for i in range(100, 120):
        y2[i] = 1

    # ---------- 画图 ----------
    plt.figure(figsize=(15, 6))
    plt.subplot(231)
    plt.plot(x[0:50], y1[0:50])
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
    # if is_periodic(y1):
    if is_periodic2(y1):
        plt.title('Periodic Signal Detected!', fontsize=9, color='r')
        plt.plot(xf1_n, yf1_n[: n // 2], 'r')
        print("y1 is periodic!!!".center(100, "-"))
    else:
        plt.title('Aperiodic Signal...', fontsize=9, color='k')
        plt.plot(xf1_n, yf1_n[: n // 2], 'k')
    plt.ylim((0, 1))

    plt.subplot(236)
    # if is_periodic(y2):
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

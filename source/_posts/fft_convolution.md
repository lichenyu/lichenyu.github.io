title: 使用FFT加速卷积求算
date: 2019/04/15
categories:
- Data Analysis
tags:
- Algorithm
---


## 原理 ##

时域卷积 <-> 频域相乘

假设序列$x1$和$x2$的长度分别为$N1$和$N2$

1. 将$x1$和$x2$通过在序列末补0方式，而使得序列的长度都为$N=(N1+N2-1)$。
2. 对$x1$和$x2$求FFT，得$f1$和$f2$。
3. 将$f1$和$f2$相乘（各个点对应相乘），得$f$。
4. 对$f$进行IFFT。


代码示例
```python
# -*- coding: UTF-8 -*-

if __name__ == '__main__':
    x1 = [1, 2, 3, 4, 5, 6, 7, 8, 9]
    x2 = [5, 6, 7]

    from numpy import convolve
    print(convolve(x1, x2))

    from numpy import zeros,real
    from numpy.fft import fft, ifft
    N = len(x1) + len(x2) - 1
    f1 = fft(x1, N)
    f2 = fft(x2, N)
    f = f1 * f2
    y = ifft(f)
    y = real(y)
    print(y)

```

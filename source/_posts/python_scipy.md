title: Python Scipy
date: 2019/07/17
categories:
- Program Development
tags:
- Python
---

## 常用方法

### FFT

```python
import numpy as np
from scipy.fftpack import fft, ifft

a = np.arange(16)

fa = fft(a)
print(fa)
'''
[120. +0.j          -8.+40.21871594j  -8.+19.3137085j   -8.+11.9728461j
  -8. +8.j          -8. +5.3454291j   -8. +3.3137085j   -8. +1.59129894j
  -8. +0.j          -8. -1.59129894j  -8. -3.3137085j   -8. -5.3454291j
  -8. -8.j          -8.-11.9728461j   -8.-19.3137085j   -8.-40.21871594j]
'''

aa = ifft(fa)
print(aa.real)
'''
[ 0.  1.  2.  3.  4.  5.  6.  7.  8.  9. 10. 11. 12. 13. 14. 15.]
'''
```

### 线性插值

#### `scipy.interpolate.interp1d(x, y, kind='linear'`
returns a function whose call method uses interpolation to find the value of new points

```python
from scipy import interpolate
import matplotlib.pyplot as plt
%matplotlib inline

x = np.arange(0, 10)
y = np.exp(-x/3.0)
f = interpolate.interp1d(x, y)

xnew = np.arange(0, 9, 0.1)
ynew = f(xnew)   # use interpolation function returned by `interp1d`
print(ynew)
plt.plot(x, y, 'o', xnew, ynew, '-')
plt.show()
```





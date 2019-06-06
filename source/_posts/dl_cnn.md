title: CNN
date: 2019/06/06
categories:
- Data Analysis
tags:
- Algorithm
---


## 图像卷积

**边缘检测 edge detection**

一般为$3 \times 3$，例如：

垂直
$$
\begin{bmatrix}
1 & 0 & -1 \\ 
1 & 0 & -1 \\ 
1 & 0 & -1
\end{bmatrix}
$$

水平
$$
\begin{bmatrix}
1 & 1 & 1 \\ 
0 & 0 & 0 \\ 
-1 & -1 & -1
\end{bmatrix}
$$

- 若其他filter内数值不同，对应不同的特性、角度
- 可以把这些数值作为参数$w$


**padding**

将原图像四周，补充一圈（对于$3 \times 3$大小的filter）

- 卷积之后，图像大小不变
- （用0padding）

**步长**

filter每次求卷积值后移动的长度，可以大于1

**图像$n \times n$，filter $f \times f$，padding $p$，步长$s$
则输出维度$floor（\frac{n+2p-f}{s} + 1）$**


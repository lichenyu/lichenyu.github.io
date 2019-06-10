title: CNN
date: 2019/06/06
categories:
- Data Analysis
tags:
- Algorithm
---


## 图像卷积

**卷积filter**

一般为$3 \times 3$，例如：

垂直边缘检测
$$
\begin{bmatrix}
1 & 0 & -1 \\ 
1 & 0 & -1 \\ 
1 & 0 & -1
\end{bmatrix}
$$

水平边缘检测
$$
\begin{bmatrix}
1 & 1 & 1 \\ 
0 & 0 & 0 \\ 
-1 & -1 & -1
\end{bmatrix}
$$

- 若其他filter内数值不同，对应不同的特性、角度
- 可以把这些数值作为参数$w$

$$
\begin{bmatrix}
w_{1,1} & w_{1,2} & w_{1,3} \\ 
w_{2,1} & w_{2,2} & w_{2,3} \\ 
w_{3,1} & w_{3,2} & w_{3,3}
\end{bmatrix}
$$

**padding**

将原图像四周，补充一圈（对于$3 \times 3$大小的filter）

- 卷积之后，图像大小不变
- （用0padding）

**步长**

filter每次求卷积值后移动的长度，可以大于1

**图像$n \times n$，filter $f \times f$，padding $p$，步长$s$，则输出维度为：$\left \lfloor \frac{n+2p-f}{s} + 1 \right \rfloor$**


**multiple filters卷积**

- 维度：$(n \times n \times n_{c}) * (f \times f \times n_{c})$ --> $((n - f + 1) \times (n - f + 1) \times n_{f})$，其中$n_{c}$为chanel数，$n_{f}$为卷积filter数
  - 使用一个$f \times f \times n_{c}$卷积，得到$((n - f + 1) \times (n - f + 1))$
  - 使用$n_{f}$个$f \times f \times n_{c}$卷积，将结果堆叠，得到$((n - f + 1) \times (n - f + 1) \times n_{f})$


## 卷积unit

**维度检查**

对于第$l$层
- $f^{[l]}$：filter大小
- $p^{[l]}$：padding大小
- $s^{[l]}$：stride大小
- input维度：$n^{[l-1]} \times n^{[l-1]} \times n_{c}^{[l-1]}$
- output维度：$n^{[l]} \times n^{[l]} \times n_{c}^{[l]}$

可知关系
- $n^{[l]} = \left \lfloor \frac{n^{[l-1]}+2p^{[l]}-f^{[l]}}{s^{[l]}} + 1 \right \rfloor$
- $n_{c}^{[l]}$：本层filter数量（$n_{c}^{[0]}$原始图像chanel数）
- 本层filter的维度：$f^{[l]} \times f^{[l]} \times n_{c}^{[l-1]}$

**activation**

- 卷积操作相当于线性$W$，进而将卷积结果进一步加$b$
- 上述结果输入一个激活函数（如ReLU）

多个filters操作、堆叠，即构成一个卷积层


## CNN各层

**conv卷积层**

**pooling**
  - 按filter抽取（如max pooling，找到各filter区域最大值）*（有点像马赛克化）*
  - 没有需要学习的参数，只需要设置超参数（filter大小、步长）

**fully connected，FC**
  - 这是NN一层

一般的，随着层数变深，图像长宽的大小$n^{[l]}$变小，但$n_{c}^{[l]}$会变大


## CNN结构

### 经典结构

conv + pooling -> conv + pooling -> fc -> fc -> softmax

### 残差网络 residual network

**residual block**

第$l$层输入，向第$l+2$层激活函数传递

$$
z^{[l+1]} = W^{[l+1]}a^{[l]} + b^{[l+1]} \\
a^{[l+1]} = g(z^{[l+1]}) \\
z^{[l+2]} = W^{[l+2]}a^{[l+1]} + b^{[l+2]} \\
a^{[l+2]} = g(z^{[l+1]} + a^{[l]})
$$


### $1 \times 1$ conv

- 控制$n_{c}$大小
- 增加非线性（使用了激活函数）


### inception network

并行堆叠conv、pooling等模块作为一层
  - $1 \times 1$ conv作为bottleneck层，加速计算














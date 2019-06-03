title: 深度学习笔记
date: 2019/06/03
categories:
- Data Analysis
tags:
- Algorithm
---


## NN基础

### 损失函数 & 成本函数 loss function & cost function

- 损失函数loss：单样本
- 成本函数cost：全体平均

$$
J(w,b) = \frac{1}{m}\sum_{l=1}^{m}L(\hat{y}^{(l)},y^{(l)})
$$


### 激活函数 activation function

- slgmold / tanh
- ReLU / leaky ReLU / P ReLU


### 梯度下降 gradient descent

寻找合适的$w$、$b$的值，能够minimize $J(w,b)$，即fit the model

1. 初始化$w$、$b$
2. 计算$dw$、$db$
3. 更新$w$、$b$
4. 迭代2、3步骤至指定次数

$$
w = w - \alpha\ dw \\
b = b - \alpha\ db
$$

根据$L$ -> $J(w,b)$，单样本 -> 全体
计算$dw$、$db$的方式：

$$
dw += x^{(l)}\ dz^{(l)},db += dz^{(l)}, \text{for }l = 1, ..., m \\
dw /= m, db /= m
$$


### 正向传播 forward propagation

**正向传播，计算结果**

单样本，第$l$层

$$
z^{[l]} = W^{[l]}a^{[l-1]} + b^{[l]} \\
a^{[l]} = g(z^{[l]})
$$

- W.shape = $(n^{[l]}, n^{[l-1]})$
- b.shape = $(n^{[l]}, 1)$
- $a^{[l-1]}$.shape = $(n^{[l-1]}, 1)$
- $z^{[l]}$.shape = $(n^{[l]}, 1)$
- $a^{[l]}$.shape = $(n^{[l]}, 1)$

全体，向量化$A^{[l]} = (a^{[l](1)}, a^{[l](2)}, , ..., a^{[l](n^{[l]})})$，第$l$层

$$
Z^{[l]} = W^{[l]}A^{[l-1]} + b^{[l]} \\
A^{[l]} = g(Z^{[l]})
$$

- $A^{[l-1]}$.shape = $(n^{[l-1]}, m)$
- $Z^{[l]}$.shape = $(n^{[l]}, m)$
- $A^{[l]}$.shape = $(n^{[l]}, m)$


### 反向传播 backward propagation

**反向传播，计算梯度**

$$
dZ^{[l]} = W^{[l+1]T}dZ^{[l+1]} * {g^{[l]}}'(Z^{[l]}) \\
dW^{[l]} = \frac{1}{m}dZ^{[l]}X^{T} \\
db^{[l]} = \frac{1}{m}np.sum(dZ^{[l]}, axls=1, keepdlms=True) \\
dA^{[l-1]} = W^{[l]T}dZ^{[l]}
$$

*简略推导（箭头反向求导，使用相乘，依据链式法则）*：

$$
Z^{[l]} \rightarrow A^{[l]} = g^{[l]}(Z^{[l]}) \rightarrow Z^{[l+1]} = W^{[l+1]}A^{[l]} + b^{[l+1]} \\
dZ^{[l+1]} \\
dA^{[l]} = W^{[l+1]}dZ^{[l+1]} \\
dZ^{[l]} = {g^{[l]}}'(Z^{[l]})dA^{[l]} = W^{[l+1]T}dZ^{[l+1]} * {g^{[l]}}'(Z^{[l]})
$$

- $dZ^{[l]}$.shape = $Z^{[l]}$.shape
- $dW^{[l]}$.shape = $W^{[l]}$.shape
- $db^{[l]}$.shape = $b^{[l]}$.shape

- 根据$J(w,b)$计算最后的$dA^{[L]}$
- 求算时，输入$dA^{[l]}$，输出$dA^{[l-1]}$

























## 模型性能

### 解决high bias

模型性能不够

- 更大更深的网络

### 解决high variance


### 正则

### dropout


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

全体，第$l$层
向量化$A^{[l]} = (a^{[l](1)}, a^{[l](2)}, , ..., a^{[l](n^{[l]})})$

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

训练误差大（比如和人工相比）
模型性能不够

- 更大更深的网络
- 增加训练时长
- （尝试其他网络结构）


### 解决high variance

训练误差小，验证/测试误差相对训练误差增大较多
模型过拟合

- 更多（多样）的数据
- 正则化
- （尝试其他网络结构）

模型可能同时存在high bias和high variance
尝试独立的先解决high bias再解决high variance


### 正则化 regularization

L2正则，解决过拟合

$$
J(w,b) + \frac{\lambda }{2m}\left \| w \right \|_{2}^{2} \\
\left \| w \right \|_{2}^{2} = \sum_{j=1}^{n_{x}}w_{j}^{2}
$$

（L1正则，使w稀疏化）
$$
J(w,b) + \frac{\lambda }{m}\left \| w \right \|_{1} \\
\left \| w \right \|_{1} = \sum_{j=1}^{n_{x}}\left | w_{j} \right |
$$

- $\lambda$作为超参数

向量化的L2正则求算
$$
J(w,b) + \frac{\lambda }{2m}\left \| w^{[l]} \right \|_{F}^{2} \\
\left \| w^{[l]} \right \|_{F}^{2} = \sum_{i=1}^{n^{l}}\sum_{j=1}^{n^{l-1}}{w_{i,j}^{[l]}}^{2}
$$

- w.shape = $(n^{[l]}, n^{[l-1]})$

如何使用

$$
dw^{[l]} = \text{from_bp} + \frac{\lambda }{m}w^{[l]} \\
w^{[l]} = w^{[l]} - \alpha \  dw^{[l]}
$$

- dw的增加项，为L2正则项的求导结果


### 随机失活 dropout

随机去除网络中一些节点

对每一层，设置该层的节点使用概率
例如，对于第3层
`d3 = np.random.rand(a3.shape[0], a3.shape[1]) < keep_prob`
`a3 = np.multiply(a3, d3)`
`a3 \= keep_prob`  <-  inverted dropout，修正期望值

使用时机：
- 训练，例如每个样本，或者每次梯度下降
- 测试，不使用dropout


## 训练性能

### 初始化$W$

使得w不太大也不太小，尽量避免梯度爆炸和消失

- ReLU：$W^{[l]} = np.random.rand(W^{[l]}.shape) * np.sqrt(\frac{2}{n^{[l-1]}})$
- tanh：$W^{[l]} = np.random.rand(W^{[l]}.shape) * np.sqrt(\frac{1}{n^{[l-1]}})$


### 梯度检查 grad check

1. 已知的$J(W, b)$看做$\theta_{i}$的函数，$J(W^{[1]}, b^{[1]}, W^{[2]}, b^{[2]}, ...) = J(\theta_{1}, \theta_{2}, \theta_{3}, \theta_{4}, ...)$
2. 已知的$dW, db$看做$d\theta_{i}$
3. 对于每一个$i$，计算$d\theta_{approx}[i]$，最终得到$d\theta_{approx}$，对比$d\theta_{approx}$和$d\theta$
$$
d\theta_{approx}[i] = \frac{J(\theta_{1}, \theta_{2}, ..., \theta_{i}+\varepsilon, ...) - J(\theta_{1}, \theta_{2}, ..., \theta_{i}-\varepsilon, ...)}{2\varepsilon} \\
d\theta_{approx} \ ?\approx \ d\theta \\
check \ \frac{\left \| d\theta_{approx} - d\theta \right \|_{2}}{\left \| d\theta_{approx} \right \|_{2} + \left \| d\theta \right \|_{2}}
$$

- 注意norm没有平方，是距离，需要差方和后开方


### mini-batch gradient descent

加速训练
将整个数据集分为若干个mini-batch，每个梯度下降迭代，使用一个mini-batch进行
这样，用整体数据集进行一次训练迭代（epoch），可进行多次梯度下降迭代（全体数据量/mini-batch大小）

typical mini-batch size: 64, 128, 256, 512


### 梯度下降替代算法

#### 背景知识：指数平滑

平滑值$v_t$

$$
v_t = \beta v_{t-1} + (1-\beta) \theta_{t}
$$

迭代获取
1. $v_{\theta} = 0$
2. `repeat: ` 获取下一个$\theta_{t}$，$v_{\theta} = \beta v_{\theta} + (1-\beta) \theta_{t}$

偏差修正（修正冷启动）：

$$
\frac{v_t}{1-\beta^{t}}
$$


#### 动量梯度下降 momentum

对梯度下降应用指数平滑（$\beta=0.9$），本质上减少了不必要的抖动，也就加速向最优值靠拢

on iteration $t$
- compute $dw$, $db$ on current mini-batch
- $v_{dw} = \beta v_{dw} + (1 - \beta)dw$
- $v_{db} = \beta v_{db} + (1 - \beta)db$
- $w = w - \alpha \ v_{dw}$
- $b = b - \alpha \ v_{db}$


#### RMSprop

比率化$dw$, $db$

on iteration $t$
- compute $dw$, $db$ on current mini-batch
- $s_{dw} = \beta s_{dw} + (1 - \beta)dw^{2}$
- $s_{db} = \beta s_{db} + (1 - \beta)db^{2}$
- $w = w - \alpha \frac{dw}{\sqrt{s_{dw}}}$
- $b = b - \alpha \frac{db}{\sqrt{s_{db}}}$


#### Adam

结合momentum、RMSprop（$\beta_{1}=0.9, \beta_{2}=0.999$）

on iteration $t$
- compute $dw$, $db$ on current mini-batch
- $v_{dw} = \beta_{1} v_{dw} + (1 - \beta_{1})dw$
- $v_{db} = \beta_{1} v_{db} + (1 - \beta_{1})db$
- $s_{dw} = \beta_{2} s_{dw} + (1 - \beta_{2})dw^{2}$
- $s_{db} = \beta_{2} s_{db} + (1 - \beta_{2})db^{2}$
- $V^{corrected}_{dw} = \frac{v_{dw}}{1 - \beta_{1}^{t}}$
- $V^{corrected}_{db} = \frac{v_{db}}{1 - \beta_{1}^{t}}$
- $S^{corrected}_{dw} = \frac{s_{dw}}{1 - \beta_{2}^{t}}$
- $S^{corrected}_{db} = \frac{s_{db}}{1 - \beta_{2}^{t}}$
- $w = w - \alpha \frac{V^{corrected}_{dw}}{\sqrt{S^{corrected}_{dw}}}$
- $b = b - \alpha \frac{V^{corrected}_{db}}{\sqrt{S^{corrected}_{db}}}$


### 学习率衰减 learning rate dacay

学习率随epoch进行而衰减

$$
\alpha = \frac{1}{1 + \text{decay_rate} \times \text{epoch_num}}
$$

title: RNN
date: 2019/06/12
categories:
- Machine Learning
tags:
- Model
---


## 序列数据

一个序列$x^{\langle 1 \rangle}, x^{\langle 2 \rangle}, ..., x^{\langle t \rangle}, ..., x^{\langle T_{x} \rangle}$，长度为$T_{x}$
- $x^{\langle t \rangle}$可使用vocabulary中的形如$[0,0,0,...,1,...,0]$的one-hot表示


## RNN结构

### 输入输出长度相同（每个时刻一个输出）

当前时刻的处理，引入了前一时刻的activation

$$
a^{\langle t \rangle} = g_{1}(W_{aa}a^{\langle t-1 \rangle} + W_{ax}x^{\langle t \rangle} + b_{a}) = g_{1}(W_{a}[a^{\langle t-1 \rangle},x^{\langle t \rangle}] + b_{a})\\
\hat{y}^{\langle t \rangle} = g_{2}(W_{y}a^{\langle t \rangle} + b_{y})
$$

- parameters第一个下标，表示该p是用于计算谁的系数；第二个下标，表示该p是谁前面的系数
- $W_{a}$为$W_{aa},W_{ax}$横向拼接；$[a^{\langle t-1 \rangle},x^{\langle t \rangle}]$为$a^{\langle t-1 \rangle},x^{\langle t \rangle}$纵向拼接
- 相当于$x^{\langle t \rangle}$位于输入层，$a^{\langle t \rangle}$位于（一层）隐藏层，$\hat{y}^{\langle t \rangle}$位于输出层

一些其他结构的RNN包括：


### many-to-one

输出只使用$\hat{y}^{\langle T_{x} \rangle}$


### one-to-many

输入只使用$\hat{x}^{\langle 1 \rangle}$


### 输入输出长度不同

两个结构拼接 encoder+decoder
前一个结构只有输入；后一个结构只有输出


## 语言模型

- 输入：$x = [0, y^{\langle 1 \rangle}, y^{\langle 2 \rangle}, ..., y^{\langle T_{y} - 1 \rangle}]$
- *softmax*，p维度为字典size
- 输出：$y = [p(\hat{y}^{\langle 1 \rangle}|0), p(\hat{y}^{\langle 2 \rangle}|y^{\langle 1 \rangle}), p(\hat{y}^{\langle 3 \rangle}|y^{\langle 1 \rangle}y^{\langle 2 \rangle}), ...]$
  - 即，given $y^{\langle 1 \rangle}, y^{\langle 2 \rangle}, ..., y^{\langle t \rangle}$，得到$y^{\langle t + 1\rangle}$的各字典项概率
  - 例如，$p(\hat{y}^{\langle 1 \rangle}|0)$为句子开头（已输入为$0$时，下一个）单词，为字典各项的概率

**sampling**

利用训好的语言模型，随机生成语句
即，将$p(\hat{y}^{\langle 1 \rangle}|0)$，作为$y^{\langle 1 \rangle}$，得到$p(\hat{y}^{\langle 2 \rangle}|y^{\langle 1 \rangle})$；继续将$p(\hat{y}^{\langle 2 \rangle}|y^{\langle 1 \rangle})$，作为$y^{\langle 2 \rangle}$...
这样得到的生成语句，体现着该语言模型的特性（例如新闻报道风格、莎士比亚文学风格...）


## cost funtion

定义各个时刻的loss，则序列总体loss为各时刻loss的累加
$$
L^{\langle t \rangle}(\hat{y}^{\langle t \rangle}, y^{\langle t \rangle}) = -y^{\langle t \rangle}\log{\hat{y}^{\langle t \rangle}} - (1-y^{\langle t \rangle})\log{(1-\hat{y}^{\langle t \rangle})} \\
L(\hat{y},y) = \sum_{t=1}^{T_{y}}L^{\langle t \rangle}(\hat{y}^{\langle t \rangle}, y^{\langle t \rangle})
$$


## 门控递归单元 get recurrent unit, GRU

解决梯度消失问题，使得较深层能够利用浅层信息

对比**naive RNN**
$$
a^{\langle t \rangle} = g_{1}(W_{aa}a^{\langle t-1 \rangle} + W_{ax}x^{\langle t \rangle} + b_{a}) = g_{1}(W_{a}[a^{\langle t-1 \rangle},x^{\langle t \rangle}] + b_{a})\\
$$
关键在于$a^{\langle t-1 \rangle}, x^{\langle t \rangle}, a^{\langle t \rangle}$，而$\hat{y}^{\langle t \rangle}$可由$a^{\langle t \rangle}$算出

而**GRU**，引入记忆细胞$c^{\langle t \rangle}$，相当于$a^{\langle t \rangle}$
由此，关键在于通过$c^{\langle t-1 \rangle}, x^{\langle t \rangle}$，如何求算$c^{\langle t \rangle}$
- 若按naive计算，$\tilde{c}^{\langle t \rangle} = g_{1}(W_{c}[c^{\langle t-1 \rangle},x^{\langle t \rangle}] + b_{c}) = tanh(W_{c}[c^{\langle t-1 \rangle},x^{\langle t \rangle}] + b_{c})$
- 或者直接透传，即本层不进行计算，直接输出$c^{\langle t-1 \rangle}$

进一步定义一个**门**，update gate, $\Gamma_{u} = \sigma(W_{u}[c^{\langle t-1 \rangle},x^{\langle t \rangle}] + b_{u})$
- 由于使用sigmoid，$\Gamma_{u}$的值，大多数情形下，或者极为接近0，或者极为接近1
- $\Gamma_{u}$决定使用按naive计算（遗忘），还是直接透传（记忆）

**计算策略**：$c^{\langle t \rangle} = \Gamma_{u} * \tilde{c}^{\langle t \rangle} + (1 - \Gamma_{u}) * c^{\langle t-1 \rangle}$
- $*$为逐元素乘
- $c^{\langle t \rangle}$是多维的，可能某些位上需要计算（遗忘），某些位上需要透传（记忆）

**完整版GRU**

$$
\Gamma_{r} = \sigma(W_{r}[c^{\langle t-1 \rangle},x^{\langle t \rangle}] + b_{r}) \\
\tilde{c}^{\langle t \rangle} = tanh(W_{c}[\Gamma_{r} * c^{\langle t-1 \rangle},x^{\langle t \rangle}] + b_{c}) \\
\Gamma_{u} = \sigma(W_{u}[c^{\langle t-1 \rangle},x^{\langle t \rangle}] + b_{u}) \\
c^{\langle t \rangle} = \Gamma_{u} * \tilde{c}^{\langle t \rangle} + (1 - \Gamma_{u}) * c^{\langle t-1 \rangle}
$$

- related gate, $\Gamma_{r}$描述$c^{\langle t-1 \rangle}$与$c^{\langle t \rangle}$之间的相关性
- 记忆细胞$c^{\langle t \rangle}$相当于$a^{\langle t \rangle}$


## 长短期记忆单元 long short term memory, LSTM

$$
\tilde{c}^{\langle t \rangle} = tanh(W_{c}[a^{\langle t-1 \rangle},x^{\langle t \rangle}] + b_{c}) \\
\Gamma_{u} = \sigma(W_{u}[a^{\langle t-1 \rangle},x^{\langle t \rangle}] + b_{u}) \\
\Gamma_{f} = \sigma(W_{f}[a^{\langle t-1 \rangle},x^{\langle t \rangle}] + b_{f}) \\
c^{\langle t \rangle} = \Gamma_{u} * \tilde{c}^{\langle t \rangle} + \Gamma_{f} * c^{\langle t-1 \rangle} \\
\Gamma_{o} = \sigma(W_{o}[a^{\langle t-1 \rangle},x^{\langle t \rangle}] + b_{o}) \\
a^{\langle t \rangle} = \Gamma_{o} * tanh(c^{\langle t \rangle})
$$

- 记忆细胞$c^{\langle t \rangle}$**不再**相当于$a^{\langle t \rangle}$
- 关键在于通过$c^{\langle t-1 \rangle}, a^{\langle t-1 \rangle}, x^{\langle t \rangle}$，如何求算$c^{\langle t \rangle}, a^{\langle t \rangle}$
- update、forget门，分别考虑需要计算（遗忘），某些位上需要透传（记忆）
- 通过output门指定，基于$c^{\langle t \rangle}$计算$a^{\langle t \rangle}$


## 双向RNN

- $a^{\langle t \rangle}$分为$\overrightarrow{a}^{\langle t \rangle}$和$\overleftarrow{a}^{\langle t \rangle}$
- 先正向计算从$\overrightarrow{a}^{\langle 1 \rangle}$至$\overrightarrow{a}^{\langle T_{x} \rangle}$；再反向计算$\overleftarrow{a}^{\langle T_{x} \rangle}$至$\overleftarrow{a}^{\langle 1 \rangle}$
- $\hat{y}^{\langle t \rangle}$由$\overrightarrow{a}^{\langle t \rangle}$和$\overleftarrow{a}^{\langle t \rangle}$算出，（如naiveRNN，$g(W_{y}[\overrightarrow{a}^{\langle t \rangle},\overleftarrow{a}^{\langle t \rangle}]) + b_{y}$）


## Deep RNN

- 隐藏层有多层，注意Deep RNN的隐藏层是要引入了前一时刻的activation的！
  - 不引入前一时刻的activation，可直接在RNN隐藏层上，堆一个NN来给出$\hat{y}^{\langle t \rangle}$（相当于RNN的hidden unit使用的一个NN）
- 由于Deep RNN的隐藏层是要引入了前一时刻的activation，系数会很多，隐藏层一般不多（3，不像Deep NN、Deep CNN，可以很深）


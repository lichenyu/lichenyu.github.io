title: seq2seq
date: 2019/06/14
categories:
- Machine Learning
tags:
- Model
---


## 基础模型

encoder（序列迭代输入） + decoder（迭代输出序列）
- encoder甚至有可能并不是一个RNN，而是例如一个CNN，对图像进行encoding


## beam search

**翻译模型，选择最可能的句子**

考虑输入$x$由encoder进行encoding，给入decoder
第一迭代，输出的$\hat{y}^{\langle 1 \rangle}$；第二迭代，输出的$\hat{y}^{\langle 2 \rangle}$...

为了得到全局最佳的句子输出，优化是不应将$\hat{y}^{\langle 1 \rangle}$、$\hat{y}^{\langle 2 \rangle}$、...分开单独考虑，而应该**全局**考虑。即优化使得given $x$，能得到最优句子$P(\hat{y}^{\langle 1 \rangle},\hat{y}^{\langle 2 \rangle},...|x)$

- 首先，翻译模型使用language model自动生成一条结果语句，每次词的选取是根据softmax的输出概率，随机sample的。这样无法保障，同一个输入，每次输出相同（预期是每次都输出最佳句子）
- 其次，若对softmax的输出，不再根据概率sample，而是直接选择概率最大的，即使用贪心算法，则如上文所述，可能得到的不是全局最佳句子。应该$(\hat{y}^{\langle 1 \rangle},\hat{y}^{\langle 2 \rangle},...|x)$，整体考虑

为此，使用beam search做最优化的选择，**流程**如下：

1. 对$\hat{y}^{\langle 1 \rangle}$，取top $B$个候选
2. 对各$\hat{y}^{\langle 1 \rangle}$候选，获取$\hat{y}^{\langle 2 \rangle}$。整体考虑前两个输出联合，可以计算$P(\hat{y}^{\langle 1 \rangle}, \hat{y}^{\langle 2 \rangle} | x) = P(\hat{y}^{\langle 1 \rangle} | x)P(\hat{y}^{\langle 2 \rangle} | x, \hat{y}^{\langle 1 \rangle})$。对于所有$\hat{y}^{\langle 1 \rangle}$候选，所有$P(\hat{y}^{\langle 1 \rangle}, \hat{y}^{\langle 2 \rangle} | x)$计算结果中，取top $B$个候选
3. 按2方式继续，直到EOS或预设的$T_{y}$

**beam search的目标函数**
$$
\text{arg }\max_{y}\prod_{t=1}^{T_{y}}P(\hat{y}^{\langle t \rangle} | x, \hat{y}^{\langle 1 \rangle}, ..., , \hat{y}^{\langle t-1 \rangle})
$$
为避免各个P过小，造成误差和无法表示（过小溢出），对P取log
$$
\text{arg }\max_{y}\sum_{t=1}^{T_{y}}\log P(\hat{y}^{\langle t \rangle} | x, \hat{y}^{\langle 1 \rangle}, ..., , \hat{y}^{\langle t-1 \rangle})
$$
但上述目标函数倾向于使用更短的输出句子，对此使用length normalization（$\alpha$为超参数）
$$
\text{arg }\max_{y}\frac{1}{T_{y}^{\alpha}}\sum_{t=1}^{T_{y}}\log P(\hat{y}^{\langle t \rangle} | x, \hat{y}^{\langle 1 \rangle}, ..., , \hat{y}^{\langle t-1 \rangle})
$$

**误差分析：RNN还是beam search有问题**

对于同一输入句子，输出：
human-level：$y^{h}$
model：$\hat{y}$

将$y^{h}$依次输入decoder，能够最终计算出$P(y^{h}|x)$

- $P(y^{h}|x) > P(\hat{y}|x)$：RNN能够计算出human-level更好，但却没选到，**beam search**有问题
- $P(y^{h}|x) < P(\hat{y}|x)$：RNN未能计算出human-level更好，但却没选到，**RNN**有问题

（若使用了length normalization，则对比length normalization之后的P）

汇总所有labeled data的对比结果，看看RNN还是beam search有问题
若RNN有问题，则进一步bias variance分析；若beam search有问题，考虑增大$B$


## attention模型

对于**seq2seq**模型，
encoder由$a^{\langle 0 \rangle} = 0$初始化，每次用$a^{\langle t'-1 \rangle}$、$x^{\langle t' \rangle}$计算$a^{\langle t' \rangle}$，最终将$a^{\langle T_{x} \rangle}$传给decoder
decoder由$a^{\langle T_{x} \rangle}$初始化，每次用$a^{\langle t-1 \rangle}$、$\hat{y}^{\langle t-1 \rangle}$计算$\hat{y}^{\langle t \rangle}$
- 计算的unit可以是naive RNN、GRU、LSTM

当输入序列过长时，decoder性能下降（记不住这么长的输入）
attention模型试图在decoder生成$\hat{y}^{\langle t \rangle}$时，引入encoder不同$t'$的$a^{\langle t' \rangle}$的考量

具体来讲，使用$s^{\langle t \rangle}$表示decoder的输出，以区分encoder的activation
$s^{\langle t \rangle}$的计算，除了输入$s^{\langle t-1 \rangle}$、$\hat{y}^{\langle t-1 \rangle}$，还增加了一个context $c^{\langle t \rangle} = \sum_{t'} \alpha^{\langle t, t' \rangle} a^{\langle t' \rangle}$
- 其中$t'$为encoder中$a$对应各时刻
- $\alpha^{\langle t, t' \rangle}$即表示计算$t$时刻的$s^{\langle t \rangle}$时，应付出多少attention在$a^{\langle t' \rangle}$上
- $\alpha^{\langle t, t' \rangle}$是归一化的，由$s^{\langle t-1 \rangle}$和$a^{\langle t' \rangle}$通过一个小NN-softmax算出


































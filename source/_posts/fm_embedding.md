title: FM embedding
date: 2019/11/08
categories:
- Machine Learning
tags:
- Model
---


## FM模型

Factorization Machines

**一般的线性模型**：
$$
\hat y(\mathbf{x}) = w_0+ \sum_{i=1}^d w_i x_i
$$

线性模型压根没有考虑特征间的关联。

- 为了表述特征间的相关性，我们采用多项式模型。
- $x_i$为第$i$个特征，$x_ix_j$（相乘，得到一个数）表示特征$x_i$和$x_j$的特征组合，其系数$w_{ij}$即为我们学习的参数

**Poly2模型**：
$$
\hat y(\mathbf{x}) = w_0+ \sum_{i=1}^d w_i x_i + \sum_{i=1}^d \sum_{j=i+1}^d w_{ij} x_i x_j
$$

但这个模型有些问题：
- 参数的个数是非常的多。$d$为特征数量（注意，one-hot后每个为单独一类，因此特征数量大大增加），对于参数数量：一次项有$d+1$个，二次项（即组合特征的参数）共有$\frac{d(d−1)}{2}$个。
- 参数与参数之间被认为彼此独立，在**稀疏场景下**（one-hot）二次项的训练是很困难的。因为要训练$w_{ij}$，需要有大量的$x_i$和$x_j$都非零的样本（只有非零组合才有意义）。而样本本身是稀疏的，满足$x_i x_j \ne 0$的样本会非常少，样本少则难以估计参数$w_{ij}$，训练出来容易导致模型的过拟合。

为解决这些问题，提出了FM模型。
上文Poly2模型，对于不同的特征对$x_i,x_j$和$x_i,x_k$认为是完全独立的，对参数$w_{ij}$和$w_{ik}$分别进行训练。
而实际上并非如此，不同的特征之间进行组合并非完全独立。

- 因此，对所有的$i$、$j$求$w_{ij}$，也就是一个矩阵$W_{ij}$时（$shape=(d,d)$），可以分解成两个矩阵相乘$\mathbf{V}^T\mathbf{V}$（$shape=(d,k)*(k,d)$）。
- 这样，在求某个$w_{ij}$时，可以分解成两个向量内积$(\mathbf{v}_i \cdot \mathbf{v}_j)$。其中，$\mathbf{v}_i$作为特征$i$的隐向量，就是$\mathbf{V}$的第$i$列，是对特征$i$的描述（可理解成类似专用于求二次项时$x_i$的权重），长度为$k$。
- 换句话说，一个（one-hot后布尔型）特征$i$，对应一个隐向量$\mathbf{v}_i$。

怎么解决上述问题的？
1）只学隐向量，参数少；2）对于几乎不出现或者很少出现的$i,j$组合，可以通过包含$i$和包含$j$的其他组合的数据，学到隐向量$\mathbf{v}_i$、$\mathbf{v}_j$就够了，反正计算时直接使用隐向量内积

**FM模型**：
$$
\hat y(\mathbf{x}) = w_0+ \sum_{i=1}^d w_i x_i + \sum_{i=1}^d \sum_{j=i+1}^d ( \mathbf{v}_i \cdot \mathbf{v}_j ) x_i x_j
$$

隐向量长度为$k\ll d$。$(\mathbf{v}_i \cdot \mathbf{v}_j)$为内积，其乘积为原来的$w_{ij}$，即$w_{ij} = ( \mathbf{v}_i \cdot \mathbf{v}_j ) = \sum_{t=1}^kv_{i,t} \cdot v_{j,t}$
二次项参数个数减少为$kd$个，$d$为特征数量（one-hot每个为一类），$k$为隐向量长度。

**FFM模型**：
$$
y(\mathbf{x}) = w_0 + \sum_{i=1}^d w_i x_i + \sum_{i=1}^d \sum_{j=i+1}^d (\mathbf{v}_{i, f_j} \cdot \mathbf{v}_{j, f_i}) x_i x_j
$$

FM模型不区分$x_i,x_j$组合和$x_k,x_j$组合中$x_j$的区别，集，使用同一个隐向量$\mathbf{v}_j$来对应特征$x_j$，与不同的$x_i$、$x_k$相乘。
而FFM则认为，对于特征$x_j$的隐向量，计算二次项系数时，对于不同的两种field组合，应该使用（各自）不同field的隐向量。
这样一来，若field有$f$个，则每个特征提供的隐向量就应该有$f$个（对应相乘特征所属field有$f$个）（而FM中只有1个哦，不区分field）。
而对于二次项参数数量，FFM模型相当于比FM模型，多了$f$倍，$f \times kd$。

**FM embedding**：
对FM模型，如果不进行最后一步的向量内积、累加，则可以认为是对$x_i$的embedding。（常数项各记录都一样，有没有无所谓）
$$
w_i x_i \qquad \mathbf{v}_i x_i \qquad \text{for all }i\text{ in } d,\quad x_i \ne 0
$$
如果$x_i$都来自于one-hot，取值为1或0，那就是
$$
w_i \qquad \mathbf{v}_i \qquad \text{for all }i\text{ in } d,\quad x_i \ne 0
$$

**为什么说FM embedding能降维**：
one-hot稀疏，但可能很长。
对于某个one-hot的数据：
- 直接做输入时，对于一个one-hot会有多个类别值，长度为其“类别数量”。
- 而进行FM embedding后，由于其只有一个“1”，输出为一个隐向量，长度为$k$。（或者算上一次项系数，那就是$k+1$）
  - 其实是one-hot特征组输入进来后，会对应“类别数量-1”个0，以及1个由“1”的位置$i$决定取值的隐向量$\mathbf{v}_i$。
  - 则相当于给定一组one-hot特征，只会对应一个特定的隐向量。

所以，对于一个one-hot，或者说对于一个field，特征维度从“类别数量”降至$k$。

即，一组同一个field的特征，对于该field的参数矩阵$\mathbf{V_{this\_field}}$只取一列。$\mathbf{V} = [\mathbf{V_{field1}}, \mathbf{V_{field2}}, \mathbf{V_{field3}}, ...]$
我们看到，降维是以field为粒度的，所以这也是后文NN结构中，对同一个field专用一个FC连接的原因。

**FM中二次项求算方法**：

对$d^2$种组合，求算$k$长度的内积，复杂度为$O(kd^2)$
下面方法，可以将复杂度降低到$O(kd)$

$$
\begin{align*}
&\sum_{i=1}^d \sum_{j=i+1}^d \langle \mathbf{v}_i, \mathbf{v}_j \rangle x_i x_j \\
=& \frac{1}{2} \sum_{i=1}^d\sum_{j=1}^d \langle \mathbf{v}_i, \mathbf{v}_j \rangle x_i x_j – \frac{1}{2}\sum_{i=1}^d \langle \mathbf{v}_i, \mathbf{v}_j \rangle x_i x_i \\
=& \frac{1}{2} \sum_{i=1}^d\sum_{j=1}^d\sum_{f=1}^k v_{i,f}v_{j,f} x_i x_j – \frac{1}{2}\sum_{i=1}^d \sum_{f=1}^k v_{i,f}v_{i,f}x_i x_i \\
=& \frac{1}{2} \sum_{f=1}^k \left( \left(\sum_{i=1}^dv_{i,f}x_i \right) \left(\sum_{j=1}^dv_{j,f}x_j \right) – \sum_{i=1}^d v_{i,f}^2x_i^2\right) \\
=&\frac{1}{2} \sum_{f=1}^k \left( \left(\sum_{i=1}^dv_{i,f}x_i \right) ^2 – \sum_{i=1}^d v_{i,f}^2x_i^2\right)
\end{align*}
$$

定义前向网络计算方式时，使用这种方法


## 利用NN的FM embedding

对每个field，连接一个（只对本field的）FC层，神经元数量即为$k$隐向量长度。
一个field，对应其下one-hot的多个特征，即，这些特征使用了一个FC层。
值得注意的是，即使各个field的维度（one-hot转化出的特征的数量）是不一样的，但是它们embedding后长度均为$k$。
- 这样做是因为，本层的参数$W$（$shape=(k,\text{n_one-hot_features}))$）相当于上文的$\mathbf{V_{this\_field}}$
- 区分各个field，是为了利用one-hot的特性，由仅有1个“1”从$\mathbf{V_{this\_field}}$只取出**一个**隐向量（“1”所在的位置，指定选哪列）（否则所有$d$个特征连接同一个FC层，会有多个“1”，会从$\mathbf{V} = [\mathbf{V_{field1}}, \mathbf{V_{field2}}, \mathbf{V_{field3}}, ...]$中取出多个隐向量，并各元素加在一起）（但如果同field中就是有多个“1”，不是one-hot，那就确实需要同field中取多个隐向量，按各元素加在一起）
- 此外，也是避免全局全连接，造成参数数量太多的问题。

**怎么训练**：
按FM公式，求预测值；数据中有真实值；梯度下降求模型参数
EM embedding时，可以锁或不锁embedding层的参数

title: 强化学习 DQN
date: 2019/11/06
categories:
- Machine Learning
tags:
- Model
---


**DQN**（Deep Q-Network），实际上就是Q Learning和Deep learning的结合，使用DNN代替Q表给出$Q$值，以解决某些场景中，状态空间或者动作空间过大（状态数、动作数过多）的问题。

价值函数近似（Value Function Approximation）的方法，通过用函数（而不是Q表）来表示$Q$值。
$$
f(s, a) = Q(s, a)
$$
为什么叫近似，因为我们并不知道$Q$值的实际分布情况，本质上就是基于训练，用一个函数来近似$Q$值的分布：
$$
f(s, a) \approx Q(s, a)
$$
这个函数可以是线性的也可以使非线性的，例如：
$$
f(s, a) = w_1s+w_2a + b
$$
当$f$为DNN时，则构成了DQN。

**值得注意**，往往求算$s$的所有$a$，故$f$的输入输出形式，除了“$f(s, a) = Q(s, a)$”，输入状态+动作，输出该状态+动作对应的$Q值$；还可以“$f(s) = [Q(s, a_1), Q(s, a_2), Q(s, a_3), ..., Q(s, a_n)]$”，输入状态，输出该状态下所有动作的$Q$值。

**那么如何训练NN呢？**

我们知道，神经网络的训练是一个最优化问题，最优化一个损失函数loss function，也就是标签和网络输出的偏差，目标是让损失函数最小化。
为此，我们需要有样本，巨量的有标签数据，然后通过反向传播使用梯度下降的方法来更新神经网络的参数。
使用什么作为标签呢？
DQN的输出是$Q$值，这个是预测结果。
我们回看Q Learning里的更新公式

$$
\begin{align*}
Q(s,a) &:= (1-\alpha) \times Q(s,a) + \alpha \times [R(s,a) + \gamma \max_{a_i}Q(s', a_i)] \\
&:= Q(s,a) + \alpha \times [R(s,a) + \gamma \max_{a_i}Q(s', a_i) - Q(s,a)]
\end{align*}
$$
即
$$
\begin{align*}
Q(s,a)_{new} &= Q(s,a)_{old} + \alpha \times [R(s,a) + \gamma \max_{a_i}Q(s', a_i) - Q(s,a)_{old}] \\
Q(s,a)_{new} - Q(s,a)_{old} &= \alpha \times [R(s,a) + \gamma \max_{a_i}Q(s', a_i) - Q(s,a)_{old}]
\end{align*}
$$
新学习时，更新的差距，在于（与$\alpha$因子相乘的）$R(s,a) + \gamma \max_{a_i}Q(s', a_i)$和$Q(s,a)_{old}$的差距。
- 将$Q(s,a)_{old}$看做当前的现状；（或者此处理解为，DNN给出的预测值）
- 将$R(s,a) + \gamma \max_{a_i}Q(s', a_i)$看做利用Reward和Q计算出来的目标Q值。把这个目标Q值作为标签，让NN给出的Q值趋近于目标Q值。（或者此处理解为，将对未来的估计当做此处的真实$Q$值，要知道真正的真实$Q$值我们是无法知道的）
- loss使用这两者mse即可

**经验回放 Experience Replay**
为什么要采用经验回放的方法？
神经网络进行训练时，假设样本是独立同分布的。而通过强化学习采集到的数据之间存在着关联性，利用这些数据进行顺序训练，神经网络当然不稳定。
经验回放通过随机抽取，可以打破数据间的关联。

具体做法是把每个时间步agent与环境交互得到的转移样本$(s_t, a_t, r_t, s_{t+1})$储存到回放记忆单元，要训练时就随机拿出一些minibatch来训练。

**目标网络 Target Net**

使用另一个网络（这里称为TargetNet）产生Target Q值。具体地，
- 当前网络MainNet的输出$Q(s,a)_{old}$或者理解为$Q(s,a)_{predict}$;
- 目标网络TargetNet的输出，代入上面求$R(s,a) + \gamma \max_{a_i}Q(s', a_i)$公式中得到Target Q值（标签）。

根据loss更新MainNet的参数，每经过N轮迭代，将MainNet的参数复制给TargetNet。

引入TargetNet后，再一段时间里Target Q值是保持不变的，一定程度降低了当前Q值和目标Q值的相关性，提高了算法稳定性。


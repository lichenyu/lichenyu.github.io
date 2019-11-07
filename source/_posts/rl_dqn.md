title: 强化学习 DQN
date: 2019/11/06
categories:
- Machine Learning
tags:
- Model
---


**DQN**（Deep Q-Network），实际上就是Q Learning和Deep Learning的结合，使用DNN代替Q表给出$Q$值，以解决某些场景中，状态空间或者动作空间过大（状态数、动作数过多）的问题。

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
- 当前网络MainNet的输出$Q(s,a)_{old}$或者理解为$Q(s,a)_{predict}$（**计算预测值**）;
- 目标网络TargetNet的输出，代入上面求$R(s,a) + \gamma \max_{a_i}Q(s', a_i)$公式中得到Target Q值（**计算标签值**）。
- （**选择$a'$的时候**，还是使用MainNet的$Q$计算并选择的）

根据loss更新MainNet的参数，每经过N轮迭代，将MainNet的参数复制给TargetNet。

引入TargetNet后，再一段时间里Target Q值是保持不变的，一定程度降低了当前Q值和目标Q值的相关性，提高了算法稳定性。

**demo**

主逻辑流程，还是Q Learning的框架
```python
def run_maze():
    step = 0
    # 迭代300回合
    for episode in range(300):
        # 初始状态
        observation = env.reset()

        while True:
            # fresh env
            env.render()

            # 根据策略，选择本次动作
            action = RL.choose_action(observation)

            # 执行动作，获取奖励、下一状态
            observation_, reward, done = env.step(action)

			# 为DQN全局存储(s_t, a_t, r_t, s_{t+1})，对应一次交互
            # 注意这些都是数值（list或单值），保存时直接flat
            RL.store_transition(observation, action, reward, observation_)

			# 更新计算Q值的DNN，200步后每隔5步训练一次
            # 训练时会抽取minibatch，梯度下降优化loss，以获取DNN的参数
            if (step > 200) and (step % 5 == 0):
                RL.learn()

            # 更新状态
            observation = observation_

            # 当前步骤为最后的结束步骤时，本回合结束，break
            if done:
                break
            step += 1

    # end of game
    print('game over')
    env.destroy()
```

DNN主要负责对MainNet和TargetNet的训练，在进入“更新”步骤时，抽取minibatch，训练两个DNN
此外还有记忆单元，全局缓存$(s_t, a_t, r_t, s_{t+1})$

```python
import numpy as np
import tensorflow as tf

np.random.seed(1)
tf.set_random_seed(1)


# Deep Q Network off-policy
class DeepQNetwork:
    def __init__(
            self,
            n_actions,
            n_features,
            learning_rate=0.01,
            reward_decay=0.9,
            e_greedy=0.9,
            replace_target_iter=300,
            memory_size=500,
            batch_size=32,
            e_greedy_increment=None,
            output_graph=False,
    ):
    	# 动作个数
        self.n_actions = n_actions
        # （一个）状态对应的特征个数，即状态的特征维度
        self.n_features = n_features
        # DNN学习率
        self.lr = learning_rate
        # gamma，计算label用
        self.gamma = reward_decay
        # 把MainNet参数拷贝至TargetNet的轮数
        self.replace_target_iter = replace_target_iter
        # 经验回放中保存的记录数
        self.memory_size = memory_size
        # DNN minibatch大小
        self.batch_size = batch_size
        # epsilon如果增加的话，最大增至多少
        self.epsilon_max = e_greedy
        # epsilon随时间增加（0到epsilon_max）
        self.epsilon_increment = e_greedy_increment
        # epsilon，策略选择的e_greedy方法用
        self.epsilon = 0 if e_greedy_increment is not None else self.epsilon_max

        # 更新DNN的当前轮数（决定是否把MainNet参数拷贝至TargetNet）
        self.learn_step_counter = 0

        # 经验回放，注意[s, a, r, s_]被flat了
        self.memory = np.zeros((self.memory_size, n_features * 2 + 2))

        # 构建前向计算OP、loss OP、train(optimizer) OP
        self._build_net()

		# 取出两个Net的参数
        t_params = tf.get_collection(tf.GraphKeys.GLOBAL_VARIABLES, scope='target_net')
        e_params = tf.get_collection(tf.GraphKeys.GLOBAL_VARIABLES, scope='eval_net')

		# 构建Net参数拷贝OP
        with tf.variable_scope('hard_replacement'):
            self.target_replace_op = [tf.assign(t, e) for t, e in zip(t_params, e_params)]

		# 启动默认图
        self.sess = tf.Session()

		# for tensorboard
        if output_graph:
            # $ tensorboard --logdir=logs
            tf.summary.FileWriter("logs/", self.sess.graph)

		# run tf变量初始化
        self.sess.run(tf.global_variables_initializer())

        # 每轮学习的cost值保存，用于画曲线
        self.cost_his = []

    def _build_net(self):
        # ------------------ all inputs ------------------------
        # 不限行，列数为特征维度
        self.s = tf.placeholder(tf.float32, [None, self.n_features], name='s')  # input State
        self.s_ = tf.placeholder(tf.float32, [None, self.n_features], name='s_')  # input Next State
        # 不限行，列数为1
        self.r = tf.placeholder(tf.float32, [None, ], name='r')  # input Reward
        self.a = tf.placeholder(tf.int32, [None, ], name='a')  # input Action

        w_initializer, b_initializer = tf.random_normal_initializer(0., 0.3), tf.constant_initializer(0.1)

        # ------------------ build evaluate_net ------------------
        # 定义MainNet前向计算输出OP
        with tf.variable_scope('eval_net'):
            e1 = tf.layers.dense(self.s, 20, tf.nn.relu, kernel_initializer=w_initializer,
                                 bias_initializer=b_initializer, name='e1')
            self.q_eval = tf.layers.dense(e1, self.n_actions, kernel_initializer=w_initializer,
                                          bias_initializer=b_initializer, name='q')

		# 定义TargetNet前向计算输出OP
        # ------------------ build target_net ------------------
        with tf.variable_scope('target_net'):
            t1 = tf.layers.dense(self.s_, 20, tf.nn.relu, kernel_initializer=w_initializer,
                                 bias_initializer=b_initializer, name='t1')
            self.q_next = tf.layers.dense(t1, self.n_actions, kernel_initializer=w_initializer,
                                          bias_initializer=b_initializer, name='t2')

		# 定义q_target OP
        with tf.variable_scope('q_target'):
            q_target = self.r + self.gamma * tf.reduce_max(self.q_next, axis=1, name='Qmax_s_')    # shape=(None, )
            # 反传截断，避免对loss进行反传时影响到TargetNet的参数
            self.q_target = tf.stop_gradient(q_target)

		# 定义q_eval OP
        with tf.variable_scope('q_eval'):
        	# 在self.a前面增加indices，生成的indices指定了哪一行取哪个动作（因为动作是0, 1, 2, ...编码的）
            a_indices = tf.stack([tf.range(tf.shape(self.a)[0], dtype=tf.int32), self.a], axis=1)
            # 收集params[indices]，即从q_eval=[Q(s, a0), Q(s, a1), Q(s, a2), ...]中取本次a对应的q
            self.q_eval_wrt_a = tf.gather_nd(params=self.q_eval, indices=a_indices)    # shape=(None, )

        # 定义loss OP
        with tf.variable_scope('loss'):
            self.loss = tf.reduce_mean(tf.squared_difference(self.q_target, self.q_eval_wrt_a, name='TD_error'))

        # 定义train (optimizer) OP
        with tf.variable_scope('train'):
            self._train_op = tf.train.RMSPropOptimizer(self.lr).minimize(self.loss)

    def store_transition(self, s, a, r, s_):
        if not hasattr(self, 'memory_counter'):
            self.memory_counter = 0
        # flat
        transition = np.hstack((s, [a, r], s_))
        # replace the old memory with new memory
        index = self.memory_counter % self.memory_size
        self.memory[index, :] = transition
        self.memory_counter += 1

    def choose_action(self, observation):
        # 在np.newaxis位置增加一个轴（因为只有一行记录）
        observation = observation[np.newaxis, :]

        if np.random.uniform() < self.epsilon:
            # MainNet执行前向计算
            actions_value = self.sess.run(self.q_eval, feed_dict={self.s: observation})
            action = np.argmax(actions_value)
        else:
            action = np.random.randint(0, self.n_actions)
        return action

    def learn(self):
        # 是否拷贝MainNet参数至TargetNet
        if self.learn_step_counter % self.replace_target_iter == 0:
            self.sess.run(self.target_replace_op)
            print('\ntarget_params_replaced\n')

        # sample batch memory from all memory
        if self.memory_counter > self.memory_size:
        	# 缓存不够，有放回抽取
            sample_index = np.random.choice(self.memory_size, size=self.batch_size)
        else:
            sample_index = np.random.choice(self.memory_counter, size=self.batch_size)
        batch_memory = self.memory[sample_index, :]

        _, cost = self.sess.run(
            [self._train_op, self.loss],
            feed_dict={
                self.s: batch_memory[:, :self.n_features], // 前特征维度个数值，为s
                self.a: batch_memory[:, self.n_features], // 1个，为a
                self.r: batch_memory[:, self.n_features + 1], // 1个，为r
                self.s_: batch_memory[:, -self.n_features:], // 后特征维度个数值，为s_next
            })

		# 记录cost
        self.cost_his.append(cost)

        # increasing epsilon
        self.epsilon = self.epsilon + self.epsilon_increment if self.epsilon < self.epsilon_max else self.epsilon_max
        self.learn_step_counter += 1

    def plot_cost(self):
        import matplotlib.pyplot as plt
        plt.plot(np.arange(len(self.cost_his)), self.cost_his)
        plt.ylabel('Cost')
        plt.xlabel('training steps')
        plt.show()

if __name__ == '__main__':
    DQN = DeepQNetwork(3,4, output_graph=True)
```

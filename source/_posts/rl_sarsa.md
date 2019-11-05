title: 强化学习 SARSA
date: 2019/11/05
categories:
- Machine Learning
tags:
- Model
---


SARSA与Q Learning类似，都是最终训练生成一个Q表。

|状态S|动作A-飞|动作A-不飞|
|---|---|---|
|$S_1$|1|20|
|$S_2$|20|-100|
|...|||
|$S_{n-1}$|-100|30|
|$S_n$|50|-20|

**[推理]**

SARSA的推理与Q Learning一样

**[训练]**

与Q Learning一样，我们会经历状态$S_1$，然后挑选一个带来最大潜在奖励的动作比如$a_2$，这样我们就到达了状态$S_2$。
在这一步，如果使用的是Q Learning，选取一个会在$S_2$上会带来最大的潜力动作，来用于计算更新Q值；但是在真正处于$S_2$上要做决定时，却不一定会选取到那个带来最大奖励的动作（$\varepsilon$-greedy）。
如果使用的是SARSA，则说到做到，在$S_2$上用于计算更新Q值的动作，也是接下来要做的动作。即，对于$Q(S_1, a_2)$的计算，不再需要使用$\max Q$，而是使用策略确定在$S_2$上选取哪个动作，比如$a_1$，然后使用$Q(S_2, a_1)$计算更新$Q(S_1, a_2)$，并最后进入$S_2$并执行$a_2$。
注意，SARSA的动作选择，只在初始化时由策略动态选择，后续各步骤迭代时，动作其实已经是确定的了。确定的时机是在上一步中，确定的方法仍是由策略给出的。

$$
Q(S,a) := (1-\alpha) \times Q(S,a) + \alpha \times [R(S,a) + \gamma Q(S', a')]
$$

**[Q值的更新（对比）]**

**SARSA算法（on-policy）**
```
Initialize Q arbitrarily // 随机初始化Q表
Repeat (for each episode): // 每一次从开始到结束是一个episode
	Initialize S // S为初始位置的状态
    Choose a from s using policy derived from Q (epsilon-greedy) // 仅第一步需使用策略选择下一步，后续步骤则在前一步中已经确定了下一步
    Repeat (for each step of episode):
        Take action a, observe r, s_next
        Choose a_next from s_next using policy derived from Q (epsilon-greedy) // 根据策略，定死了下一步选哪个
        Q(s, a) := (1 - ALPHA) * Q(s, a) + ALPHA * (r + GAMMA * Q(s_next, a_next)) // 更新Q(s, a)
        s := s_next; a := a_next // 更新s、a
    until s is terminal // 到游戏结束为止，即到达最后一步
```

处于状态`s`时，1）若是第一步则根据策略来选取动作`a`；2）若是中间步骤则使用上一步中指定的`a`。进而观测到下一步状态`s_next`，并再次根据策略选择动作`a_next`，这样就有了一个`[s, a, r, s_next, a_next]`序列（SARSA）。
处于状态`s_next`时，在上一步就知道了要采取哪个`a_next`，此时真的采取`a_next`这个动作...
动作的选取遵循相同的策略（比如epsilon-greedy）。

**Q-learning算法（off-policy）**
```
Initialize Q arbitrarily // 随机初始化Q表
Repeat (for each episode): // 每一次从开始到结束是一个episode
    Initialize S // S为初始位置的状态
    Repeat (for each step of episode):
        Choose a from s using policy derived from Q (epsilon-greedy) // 每次都是根据当前S和Q，使用一种策略，动态得到（下一步）
        Take action a, observe r, s_next
        Q(s, a) := (1 - ALPHA) * Q(s, a) + ALPHA * (r + GAMMA * Q(s_next, a_maxQ_at_snext)) // 更新Q(s, a)
        s := s_next // 只更新s，a由下一步动态确定
    until s is terminal // 到游戏结束为止，即到达最后一步
```

处于状态`s`时，根据策略来选取动作`a`。进而观测到下一步状态`s_next`，这样就有了一个`[s, a, r, s_next]`序列。
处于状态`s_next`时，再次根据策略来选取动作`a_next`。

**[SARSA lambda]**

对$Q$值更新，不只更新本次获取$r$的步骤，而是更新之前各个步骤，不过使用$\lambda$来衰减。

```
Initialize Q arbitrarily // 随机初始化Q表
Repeat (for each episode): // 每一次从开始到结束是一个episode
    for all s in S, a in A: E(s, a) = 0 // 初始化E表
	Initialize S // S为初始位置的状态
    Choose a from s using policy derived from Q (epsilon-greedy) // 仅第一步需使用策略选择下一步，后续步骤则在前一步中已经确定了下一步
    Repeat (for each step of episode):
        Take action a, observe r, s_next
        Choose a_next from s_next using policy derived from Q (epsilon-greedy) // 根据策略，定死了下一步选哪个
        delta = r + GAMMA * Q(s_next, a_next) - Q(s, a)
        E(s, a) := E(s, a) + 1 // 更新E表
        for all s in S, a in A: // 对E表中所有点
        	Q(s, a) += E(s, a) * ALPHA * delta // 利用更新后的E表，计算更新Q(s, a)
            E(s, a) := LAMBDA * E(s, a) + 1 // 衰减E表
        s := s_next; a := a_next // 更新s、a
    until s is terminal // 到游戏结束为止，即到达最后一步
```

先定义一个$E$表，用来记录经过的各个位置（$S$）。
每走一步，如果这个点不在$E$表中则添加这个点到$E$表中，更新$E(s,a)$的值改为$E(s,a)+1$（还可以优化，下面说）。
而在走下一步之前，会对$E$表进行整体衰减。
也就是说每走一步，就要对$E$表的当前位置的值进行刷新，然后计算$Q$表，然后衰减$E$表。
衰减的意义就在于如果一旦到达终点，就可以体现出来$E$表中各个位置对到达终点的不可或缺性。
- 如果衰减比例为0，也就是每次都给$E$表里的值乘0，就意味着表里只会剩下最后一个位置，这就是上文的SARSA；
- 如果衰减比例为1，则不会出现衰减，完全保留并更新之前经过的所有$S$。但有一个问题：$E$表里的重复的越多的位置收益越大。前面我们说每次经过这个某个位置，都把$E$表里对应值+1，这样对有些位置会很不公平，可能会出现离终点最近的那个位置的$E$值反而比中间的某个点的$E$值要低，这很不科学。优化办法就是给$E$里的值定个上限，比如就是1，每次走到这个位置，就把他重新定为1，然后从1开始衰减，这样就不会出现上述的问题了。

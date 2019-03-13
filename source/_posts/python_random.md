title: Python - 随机数获取示例
date: 2019/03/13
categories:
- Program Development
tags:
- Python
---


## import random ##

```python
# [0.0, 1.0)
random.random()

# [a, b]
random.randint(a, b)

# 返回float类型，b不一定被包含
random.uniform(a, b)

# [start, stop)
random.randrange(start, stop[, step])

# 返回一个随机元素
random.choice(seq)

# 洗牌
random.shuffle(x[, random])

# 不放回采样k个不同元素（如需有放回采样，重复做choice即可）
random.sample(population, k)
```

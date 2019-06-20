title: Tensorflow Cheatsheet
date: 2019/06/20
categories:
- Program Development
tags:
- Tensorflow
---


### 定义OP、执行OP

```python
# ---------- 定义OP----------
m1 = tf.constant([[3, 3]])  # 常量OP
m2 = tf.constant([[2], [3]])
product = tf.matmul(m1, m2)  # 计算OP

state = tf.Variable(0, name='counter')  # 变量OP
get_new_value = tf.add(state, 1)
update = tf.assign(state, get_new_value)  # 为变量赋值OP

# 为变量定义一个初始化OP：变量执行时，需要先初始化
init = tf.global_variables_initializer()


# ---------- 执行OP ----------
with tf.Session() as sess:
	# 总是sess.run一个OP，才会产生（计算）结果
	res = sess.run(product)
	print(res)
	
	sess.run(init)
	print(sess.run(state))
	for _ in range(5):
		sess.run(update)
	print(sess.run(state))
```

### feed

```python
input1 = tf.placeholder(tf.float32)
input2 = tf.placeholder(tf.float32)
output = tf.multiply(input1, input2)

with tf.Session() as sess:
	res = sess.run(output, feed_dict={intput1:[7.], input2:[2.]})
	print(res)
```


### training通用流程

```python
# prediction为之前基于x（placeholder），已定义的预测输出
# y为ground truth（placeholder）

# 定义cost function OP
loss = tf.reduce_mean(tf.square(y - prediction))
# 定义优化方法（一次梯度下降迭代）OP
train_step = tf.train.GradientDescentOptimizer(0.01).minimize(loss)

with tf.Session() as sess:
	sess.run(tf.global_variables_initializer())
	for _ in range(2000):
		# 每次优化迭代，改变prediction计算中的Variables，降低loss
		sess.run(train_step, feed_dict={x:x_data, y:y_data})


	# 预测时只需
	sess.run(pridiction, feed_dict={x:x_data}

```



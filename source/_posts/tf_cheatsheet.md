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


### mini batch

```python

batch_size = 100
n_batch = m // batch_size

#...

with tf.Session as sess:
	# 迭代2000个epoch
	for epoch in range(2000):
		# 每个epoch有n_batch个batch
		for batch in range(n_batch):
			# 获取batch_size大小的batch
			batch_x, batch_y = get_next_batch()
			sess.run(train_step, feed_dict={x:batch_x, y:batch_y})
```


### dropout

```python
keep_prob = tf.placeholder(tf.float32)

layer1 = tf.nn.relu(tf.matmul(W1, x) + b1)
layer1 = tf.nn.dropout(layer1, keep_prob=keep_prob)

layer2 = tf.nn.relu(tf.matmul(W2, layer1) + b2)
layer2 = tf.nn.dropout(layer2, keep_prob=keep_prob)

```


### 初始化

随机初始化
- W
  - `tf.Variable(tf.truncated_normal(shape, stddev=0.1))`
- b
  - `tf.Variable(tf.zeros(shape) + 0.1)`

**note**：好的初始化系数、优化器等设置，可以在迭代次数一定的情形下，更好的逼近最优解，从而提升模型性能


--- 

### 计算准确率（OP）

```python
correct_prediction = tf.equal(tf.argmax(y, 1), tf.argmax(prediction, 1))  # tf.argmax返回最大值下标
accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))

# ...

with tf.Session as sess:
	
	# ...
	
	acc = sess.run(accuracy, feed_dict(x:test_x, y:test_y))
	print(acc)
```


--- 

### tensorboard

```python
with tf.name_scope('input'):
	input1 = tf.placeholder(tf.float32, [nx, none], name='input1')
	input2 = tf.placeholder(tf.float32, [nx, none], name='input2')
	

tf.summary.scalar('name', var)

# ...

merged = tf.summary.merge_all()

with tf.Session() as sess:
	writer = tf.summary.FileWriter('dir', sess.graph)
	
	# for each epoch
	summary = sess.run(merged)
	writer.add_summary(summary, epoch)
```

- `tensorboard --logdir=dir`
- name_scope在定义图时进行设置（执行图则仅对定义图进行计算）


### 模型存取

模型保存
```python
saver = tf.train.Saver()
# ...
saver.save(sess, 'file_path/file_name')
```

模型读取
```python
saver = tf.train.Saver()
# ...
saver.restore(sess, 'file_path/file_name')
```
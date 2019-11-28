title: Weight Initialization
date: 2019/11/28
categories:
- Machine Learning
tags:
- Model
---


## 权重初始化方法

策略：

|Activation Function|Uniform Distribution $[-r, +r]$|Normal Distribution||
|---|---|---|---|
|sigmoid|$r = 4\sqrt{\frac{6}{n_{in} + n_{out}}}$|$\sigma = 4\sqrt{\frac{2}{n_{in} + n_{out}}}$|Glorot|
|tanh|$r = \sqrt{\frac{6}{n_{in} + n_{out}}}$|$\sigma = \sqrt{\frac{2}{n_{in} + n_{out}}} = \sqrt{\frac{1}{n_{in}}}$|Glorot|
|relu|$r = \sqrt{2}\sqrt{\frac{6}{n_{in} + n_{out}}}$|$\sigma = \sqrt{\frac{4}{n_{in} + n_{out}}}  = \sqrt{\frac{2}{n_{in}}}$|He|

示例：

```python
glorot = np.sqrt(2.0 / (self.deep_layers[i - 1] + self.deep_layers[i]))
weights["layer_%d" % i] = tf.Variable(
	np.random.normal(loc=0, scale=glorot, size=(self.deep_layers[i - 1], self.deep_layers[i])),
    dtype=np.float32)
```
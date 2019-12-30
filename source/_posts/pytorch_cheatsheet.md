title: PyTorch Cheatsheet
date: 2019/12/25
categories:
- Program Development
tags:
- PyTorch
---


#### Tensor

```python
import torch
x = torch.Tensor(5, 3)  # 构造一个未初始化的5*3的矩阵
x = torch.rand(5, 3)  # 构造一个随机初始化的矩阵
x # 此处在notebook中输出x的值来查看具体的x内容
x.size()

#NOTE: torch.Size 事实上是一个tuple, 所以其支持相关的操作*
y = torch.rand(5, 3)

#此处 将两个同形矩阵相加有两种语法结构
x + y # 语法一
torch.add(x, y) # 语法二

# 另外输出tensor也有两种写法
result = torch.Tensor(5, 3) # 语法一
torch.add(x, y, out=result) # 语法二
y.add_(x) # 将y与x相加，inplace

# 特别注明：任何可以改变tensor内容的操作都会在方法名后加一个下划线'_'
# 例如：x.copy_(y), x.t_(), 这俩都会改变x的值。

#另外python中的切片操作也是资次的。
x[:,1] #这一操作会输出x矩阵的第二列的所有值

```

#### Tensor与numpy

```python
# 此处演示tensor和numpy数据结构的相互转换
a = torch.ones(5)
b = a.numpy()

# 此处演示当修改numpy数组之后,与之相关联的tensor也会相应的被修改
a.add_(1)
print(a)
print(b)

# 将numpy的Array转换为torch的Tensor
import numpy as np
a = np.ones(5)
b = torch.from_numpy(a)
np.add(a, 1, out=a)
print(a)
print(b)

# 另外除了CharTensor之外，所有的tensor都可以在CPU运算和GPU预算之间相互转换
# 使用CUDA函数来将Tensor移动到GPU上
# 当CUDA可用时会进行GPU的运算
if torch.cuda.is_available():
    x = x.cuda()
    y = y.cuda()
    x + y
```

#### Autograd

```python
from torch.autograd import Variable
x = Variable(torch.ones(2, 2), requires_grad = True)
y = x + 2
y.creator

# y 是作为一个操作的结果创建的因此y有一个creator
z = y * y * 3
out = z.mean()

# 现在我们来使用反向传播
out.backward()

# out.backward()和操作out.backward(torch.Tensor([1.0]))是等价的
# 在此处输出 d(out)/dx
x.grad
```

#### Neural Network

- 定义一个有着可学习的参数（或者权重）的神经网络
- 对着一个输入的数据集进行迭代:
  - 用神经网络对输入进行处理
  - 计算代价值 (对输出值的修正到底有多少)
  - 将梯度传播回神经网络的参数中
  - 更新网络中的权重
    - 通常使用简单的更新规则: weight = weight + learning_rate * gradient

定义网络
```python
import torch.nn as nn
import torch.nn.functional as F

class LRModel(torch.nn.Module):
    def __init__(self):
        super(Model, self).__init__()
        # linear内置了参数初始化方法，此处无需显式指定
        self.linear = torch.nn.Linear(2, 1) # 2 in, 1 out

    def forward(self, x):
        y_pred = F.sigmoid(self.linear(x))
        return y_pred
```

仅仅需要定义一个forward函数就可以了，backward会自动地生成。
你可以在forward函数中使用所有的Tensor中的操作。
模型中可学习的参数会由net.parameters()返回。

定义loss与optimizer
```python
criterion = torch.nn.BCELoss()
optimizer = torch.optim.SGD(model.parameters(), lr=0.01)
```

训练迭代
```python
x_data = Variable(torch.Tensor(X))
y_data = Variable(torch.Tensor(T))

model = LRModel()

for epoch in range(1000):
    # Forward pass: Compute predicted y by passing x to the model
    y_pred = model(x_data)

    # Compute and print loss
    loss = criterion(y_pred, y_data)
    print(epoch, loss.data[0])

    # Zero gradients, perform a backward pass, and update the weights.
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()
```























































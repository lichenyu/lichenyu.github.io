title: Python Numpy
date: 2019/07/15
categories:
- Program Development
tags:
- Python
---


## ndarray 对象 ##

同类型元素的N维数组

### 常用创建

#### `np.array(object)`、`np.asarray(object)`

由object（常见为list）创建ndarray数组对象

#### `np.empty(shape, dtype)`

创建一个指定形状、数据类型、未初始化的数组
注意，empty指的是未初始化，数组元素值为**随机值**，并不是0值

#### `np.zeros(shape, dtype)`、`np.ones(shape, dtype)`

创建一个指定形状、指定数据类型，由0或1填充的数组

#### `np.arange(start, stop, step, dtype)`

根据`start`与`stop`指定的范围，以及`step`设定的步长，生成一个指定类型的数组。

#### `np.linspace(start, stop, num=50, endpoint=True, retstep=False)`

根据`start`、`stop`、`num`生成等差数列数组，`endpoint`表明是否包含`stop`值，`retstep`表明结果是否返回公差。

#### `np.logspace(start, stop, num=50, endpoint=True, base=10.0)`

根据`start`、`stop`、`num`生成等比数列数组，注意：序列起始值为`base ** start`，序列的终止值为：`base ** stop`。

#### `np.random.rand(d0, d1, ..., dn)`

随机值`[0, 1)`
`d`给出shape


### 常用属性

#### 切片

一维
- `arr[start:stop:step]`
- `arr[start:]`，start直至最后

二维
- `arr[:, 1]`，所有行、第二列
- `arr[..., 1]`，所有行、第二列
- 对于多维，如3darray，`arr[..., 1] == arr[:, :, 1]`
- `arr[..., 1:]`，所有行、第二至最后列
- `arr[[0,1,2], [0,1,0]]`，分别指定行、列。即，`arr[rows,cols]`，rows、cols为array，下标各自设定

布尔索引
- `arr[arr >  5]`，其中`arr > 5`会给出一个和arr同shape的布尔索引

花式索引
- `arr[[4, 2, 1, 7]]`，索引为数组。如果目标是一维数组，那么索引的结果就是对应位置的元素；如果目标是二维数组，那么就是对应下标的行。

**注意**：切片数组返回一个view，而非copy；这与python list不同，python list的切片是copy

#### `ndarray.ndim`

轴的数量，或维度的数量。轴编号从`0`到`ndim-1`。

很多操作可以声明`axis`。`axis=i`，代表沿着第`i`个轴（下标变化）的方向，进行操作。
例如，对于**二维数组**：`axis=0`表示**沿着第0轴**进行操作，即固定其他轴，遍历对第一个下标，这样相当于每次处理了一列的数据，即按**列**操作；`axis=1`表示**沿着第1轴**进行操作，即按**行**操作。

更加具体的例子：
```python
arr = np.arange(16).reshape(2, 4, 2)
'''
array([[[ 0,  1],
        [ 2,  3],
        [ 4,  5],
        [ 6,  7]],

       [[ 8,  9],
        [10, 11],
        [12, 13],
        [14, 15]]])
'''
arr.sum(axis=0)
'''
array([[ 8, 10],
       [12, 14],
       [16, 18],
       [20, 22]])
'''
```
例子中，`arr`的`shape`为`(2, 4, 2)`，`arr`的`shape`的下标为`(0, 1, 2)`。
`axis=0`，即对`shape`下标的第一个位置进行处理。由`(2, 4, 2)`可知，该位置变化方向有2个，最后结果`shape`为`(4, 2)`。
求算时，`arr[0][a][b]`+`arr[1][a][b]`作为结果`s[a][b]`的值。

#### `ndarray.shape`

数组各维度的大小

#### `ndarray.size`

数组元素的总个数（等于shape中各值相乘）

#### `ndarray.dtype`

数组的元素类型

#### `ndarray.flat`

数组元素迭代器


### 常用方法

#### 调整形状

#### `ndarray.reshape()`、`np.resize(arr, shape)`

调整数组shape，返回view或copy

#### `ndarray.ravel()`、`ndarray.flatten()`

返回一份数组flat后的view或copy

#### 翻转

#### `ndarray.T`、`np.transpose(arr, axes)`

转置数组，axes指定转置后的维度下标顺序
如3darray，默认axes为[0, 1, 2]，当指定为[1, 0, 2]时，所有元素第一下标和第二下标互换位置

#### 增删维度

#### `np.expand_dims(arr, axis)`

在指定位置插入新的轴来扩展数组arr形状，`axis`为新轴插入的位置
例如，一个shape为(2, 2)的数组，经过`axis=0`扩展，新shape为(1, 2, 2)

#### `np.squeeze(arr, axis)`

从给定数组的形状中删除一维的条目
例如，shape(1, 2, 2) -> shape(2, 2)

#### 拼接

#### `np.concatenate((a1, a2, ...), axis)`
#### `np.vstack()` `np.r_[ar1, ar2]`
#### `np.hstack()` `np.c_[ar1, ar2]`
#### `np.dstack()`
#### `np.stack(arrays, axis)` **（会拓展维度）**

```python
a = np.zeros((2, 3), dtype=np.uint64)
'''
[[0 0 0]
 [0 0 0]]
'''
b = np.ones((2, 3), dtype=np.uint64)
'''
[[1 1 1]
 [1 1 1]]
'''
# a、b的shape可以不完全相同，但需要在待拼接的维度中一致

c1 = np.concatenate((a, b))
c11 = np.vstack((a, b)) #np.r_
'''
[[0 0 0]
 [0 0 0]
 [1 1 1]
 [1 1 1]]
'''
c2 = np.concatenate((a, b), axis=1)
c22 = np.hstack((a, b)) #np.c_
'''
[[0 0 0 1 1 1]
 [0 0 0 1 1 1]]
'''

# np.stack会新创建一个维度，位置由axis指定；而上文三个方法不会新增维度
cc1 = np.stack((a, b))
'''
[[[0 0 0]
  [0 0 0]]

 [[1 1 1]
  [1 1 1]]]
'''
print(cc1.shape)
# (2, 2, 3)

# 深度方向堆叠
cc2 = np.stack((a, b), axis=2) #np.dstack()
'''
[[[0 1]
  [0 1]
  [0 1]]

 [[0 1]
  [0 1]
  [0 1]]]
'''
print(cc2.shape)
# (2, 3, 2)
```

**增加行（对行进行拼接）的方法有：**

```python
np.concatenate((ar1, ar2),axis=0)
np.append(ar1, ar2, axis=0)
np.vstack((ar1, ar2))
np.row_stack((ar1, ar2))
np.r_[ar1,ar2]
```

**增加列（对列进行拼接）的方法有：**

```python
np.concatenate((ar1, ar2),axis=1)
np.append(ar1, ar2, axis=1)
np.hstack((ar1, ar2))
np.column_stack((ar1, ar2))
np.c_[ar1,ar2]
```

**增加维度进行拼接的方法有：**

```python
np.stack((ar1, ar2), axis=-1)
np.dstack((ar1, ar2))
```
或者用`np.expand_dims`拓展维度`axis=0`后，对行进行拼接，然后`transpose`

#### 拆分

#### `np.split(ary, indices_or_sections, axis)`

若参数`indices_or_sections`是一个整数，则为分割区间数量
若参数`indices_or_sections`是一个数组，则为分割点，分割区间左开右闭

#### `np.vsplit()`
#### `np.hsplit()`

```python
a = np.arange(24).reshape(4, 6)
'''
[[ 0  1  2  3  4  5]
 [ 6  7  8  9 10 11]
 [12 13 14 15 16 17]
 [18 19 20 21 22 23]]
'''

b = np.split(a, 2)
b = np.vsplit(a, 2)
'''
[array([[ 0,  1,  2,  3,  4,  5],
       [ 6,  7,  8,  9, 10, 11]]), array([[12, 13, 14, 15, 16, 17],
       [18, 19, 20, 21, 22, 23]])]
'''
b = np.split(a, 3, axis=1)
b = np.hsplit(a, 3)
'''
[array([[ 0,  1],
       [ 6,  7],
       [12, 13],
       [18, 19]]), array([[ 2,  3],
       [ 8,  9],
       [14, 15],
       [20, 21]]), array([[ 4,  5],
       [10, 11],
       [16, 17],
       [22, 23]])]
'''
# 若参数是一个数组，则为分割点，分割区间左开右闭
b = np.split(a, [1, 3])
'''
[array([[0, 1, 2, 3, 4, 5]]), array([[ 6,  7,  8,  9, 10, 11],
       [12, 13, 14, 15, 16, 17]]), array([[18, 19, 20, 21, 22, 23]])]
'''
```

#### 元素操作

#### `np.append(arr, values, axis=None)`

数组的末尾添加值
当`axis`无定义时，返回总是为一维数组
当`axis`有定义时，待添加的元素需要包含指定轴以下各级

```python
a = np.arange(24).reshape(4, 6)
'''
[[ 0  1  2  3  4  5]
 [ 6  7  8  9 10 11]
 [12 13 14 15 16 17]
 [18 19 20 21 22 23]]
'''

b = np.append(a, [24, 25])
'''
[ 0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23
 24 25]
'''

# append元素shape需覆盖[new, 0:]
b = np.append(a, [[24, 25, 26, 27, 28, 29]], axis=0)
'''
[[ 0  1  2  3  4  5]
 [ 6  7  8  9 10 11]
 [12 13 14 15 16 17]
 [18 19 20 21 22 23]
 [24 25 26 27 28 29]]
'''

# append元素shape需覆盖[0:, new]
b = np.append(a, [[24], [25], [26], [27]], axis=1)
'''
[[ 0  1  2  3  4  5 24]
 [ 6  7  8  9 10 11 25]
 [12 13 14 15 16 17 26]
 [18 19 20 21 22 23 27]]
'''
```

#### `np.insert(arr, obj, values, axis)`

在指定索引之前，沿指定轴，向数组中插入值
`axis=1`时，`obj`使用`[]`指定索引，否则会被广播

```python
a = np.arange(24).reshape(4, 6)
'''
[[ 0  1  2  3  4  5]
 [ 6  7  8  9 10 11]
 [12 13 14 15 16 17]
 [18 19 20 21 22 23]]
'''

b = np.insert(a, 2, [88, 99])
'''
[ 0  1 88 99  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21
 22 23]
'''

# [new, 0:]
b = np.insert(a, 2, [[24, 25, 26, 27, 28, 29]], axis=0)
'''
[[ 0  1  2  3  4  5]
 [ 6  7  8  9 10 11]
 [24 25 26 27 28 29]
 [12 13 14 15 16 17]
 [18 19 20 21 22 23]]
'''

# [0:, new]
b = np.insert(a, [2], [[24], [25], [26], [27]], axis=1)
'''
[[ 0  1 24  2  3  4  5]
 [ 6  7 25  8  9 10 11]
 [12 13 26 14 15 16 17]
 [18 19 27 20 21 22 23]]
'''

b = np.insert(a, 2, [[24], [25], [26], [27]], axis=1)
'''
[[ 0  1 24 25 26 27  2  3  4  5]
 [ 6  7 24 25 26 27  8  9 10 11]
 [12 13 24 25 26 27 14 15 16 17]
 [18 19 24 25 26 27 20 21 22 23]]
'''
```

#### `np.delete(arr, obj, axis)`

在指定索引，沿指定轴，删除数组中值

```python
a = np.arange(24).reshape(4, 6)
'''
[[ 0  1  2  3  4  5]
 [ 6  7  8  9 10 11]
 [12 13 14 15 16 17]
 [18 19 20 21 22 23]]
'''

b = np.delete(a, 2)
'''
[ 0  1  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23]
'''

b = np.delete(a, 2, axis=0)
'''
[[ 0  1  2  3  4  5]
 [ 6  7  8  9 10 11]
 [18 19 20 21 22 23]]
'''

b = np.delete(a, 2, axis=1)
'''
[[ 0  1  3  4  5]
 [ 6  7  9 10 11]
 [12 13 15 16 17]
 [18 19 21 22 23]]
'''
```

#### `np.unique(arr, return_index=False, return_inverse=False, return_counts=False)`

去除数组中的重复元素
- `return_index`是否同时返回新列表元素在旧列表的下标
- `return_inverse`是否同时返回旧列表元素在新列表的下标
- `return_counts`是否同时返回新列表元素在旧列表的出现次数

#### 数学舍入

#### `np.around(a, decimals)`
四舍五入，`decimals`指定保留到小数点后哪一位，当取负值时，将四舍五入推广到十位、百位...等相应位置
#### `np.floor(a)`
#### `np.ceil(a)`

#### 数学计算

#### `np.reciprocal(a)`
取倒数
#### `np.power(a, b)`
a ** b
#### `np.mod(a)`
取余

#### 统计

#### `np.amin(a)`、`np.nanmin(a)`
#### `np.amax(a)`、`np.nanmax(a)`
#### `np.sum(a)`、`np.nansum(a)`
#### `np.percentile(a, q)`、`np.nanpercentile(a, q)`
q在0~100之间
#### `np.quantile(a, q)`、`np.nanquantile(a, q)`
q在0~1之间
#### `np.median(a)`、`np.nanmedian(a)`
#### `np.mean(a)`、`np.nanmean(a)`
#### `np.std(a)`、`np.nanstd(a)`
#### `np.var(a)`、`np.nanvar(a)`

#### 线性代数

#### `np.dot(a, b)`
矩阵乘法
#### `np.inner(a)`
向量内积（对位相乘后相加）

#### 排序

#### `np.sort(a)`
#### `np.argsort(a)`
从小到大的排序后a的索引值
#### `np.argmax(a)`、`np.argmin(a)`
最大和最小元素的索引

#### 选择

#### `np.nonzero(a)`
#### `np.where(c)`

#### 深拷贝

#### `ndarray.copy()`


title: Python Numpy
date: 2019/07/15
categories:
- Program Development
tags:
- Python
---


## ndarray 对象 ##

同类型元素的N维数组

### 创建

|方法|说明|
|---|---|
|`np.array(object)`|由object（常见为list）创建ndarray数组对象|
|`np.asarray(object)`|由object（常见为list）创建ndarray数组对象|
||由list创建时，二者一样；由ndarray创建时，array会新复制出一个ndarray，asarray则仍引用原来的ndarray|
|`np.empty(shape, dtype)`|创建一个指定形状、数据类型、未初始化的数组|
||注意，empty指的是未初始化，数组元素值为**随机值**，并不是0值|
|`np.zeros(shape, dtype)`|创建一个指定形状、指定数据类型，由0填充的数组|
|`np.ones(shape, dtype)`|创建一个指定形状、指定数据类型，由1填充的数组|
|`np.arange(start, stop, step, dtype)`|根据`start`、`stop`、`step`，生成一个指定类型的定步长数组|
|`np.linspace(start, stop, num=50, endpoint=True, retstep=False)`|根据`start`、`stop`、`num`，生成一个等差数列数组，`endpoint`表明是否包含`stop`值，`retstep`表明结果是否返回公差|
|`np.logspace(start, stop, num=50, endpoint=True, base=10.0)`|根据`start`、`stop`、`num`，生成一个等比数列数组，序列起始值为`base ** start`，序列的终止值为：`base ** stop`|
|`np.random.rand(d0, d1, ..., dn)`|随机值`[0, 1)`序列，`d`给出shape|
|`ndarray.copy()`|深拷贝|

### 属性

|方法|说明|
|---|---|
|`ndarray.shape`|数组各维度的大小（是数量不是阶数）|
|`ndarray.size`|数组元素的总个数（等于shape中各值相乘）|
|`ndarray.dtype`|数组的元素类型|
|`ndarray.ndim`|轴的数量，或维度的数量。轴编号从`0`到`ndim-1`|

**轴的概念**

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
`axis=0`，即对`shape`下标的第一个位置进行处理。由`(2, 4, 2)`可知，每次处理2个，固定`(4, 2)`作为最后结果`shape`。
求算时，`arr[0][a][b]`+`arr[1][a][b]`作为结果`s[a][b]`的值。

### 切片

**注意**：切片数组返回一个view，而非copy；这与python list不同，python list的切片是copy

|方法|说明|
|---|---|
|`arr[start:stop:step]`|一维|
|`arr[start:]`|一维，start直至最后|
|`arr[:, 1]`|二维，所有行、第二列|
|`arr[1, :]`|二维，第二行、所有列|
|`arr[..., 1]`|二维，所有行、第二列|
||对于多维，如3darray，`arr[..., 1] == arr[:, :, 1]`|
|`arr[..., 1:]`|二维，所有行、第二至最后列|
|`arr[0:1, 1:2]`|二维，一至二行、二至三列|
|`arr[[0,1,2], [0,1,3]]`|二维，分别指定行、列。即，`arr[rows,cols]`|
|`arr[arr >  5]`|布尔索引，其中`arr > 5`会给出一个和arr同shape的布尔数组，按这个布尔数组索引|
|`arr[[4, 2, 1, 7]]`|花式索引，索引为数组。如果目标是一维数组，那么索引的结果就是对应位置的元素；如果目标是二维数组，那么就是对应下标的行。

数字后的`:`表示范围；单独的`:`表示所有

### 调整形状

|方法|说明|
|---|---|
|`ndarray.reshape()`|调整数组shape，返回view|
|`np.resize(arr, shape)`|调整数组shape，返回copy|
|`ndarray.flat`|返回数组元素迭代器（可直接索引`arr.flat[3]`）|
|`ndarray.ravel()`|返回数组flat后的view|
|`ndarray.flatten()`|返回数组flat后的copy（默认参数为"C"，即按照行flat；设置参数为"F"，可按照列falt）|
|`ndarray.T`|转置数组|
|`np.transpose(arr, axes)`|转置数组，axes指定转置后的维度下标顺序。如3darray，默认axes为[0, 1, 2]，当指定为[1, 0, 2]时，所有元素第一下标和第二下标互换位置|
|`np.expand_dims(arr, axis)`|扩展维度，`axis`为新轴插入的位置。例如，一个shape为(2, 2)的数组，经过`axis=0`扩展，新shape为(1, 2, 2)|
|`np.squeeze(arr, axis)`|从给定数组，把shape中为1的维度去掉。例如，shape(1, 2, 2) -> shape(2, 2)。可通过`axis`参数指定需要删除的维度，但是指定的维度必须为单维度，否则将会报错|

### 拼接

|方法|说明|
|---|---|
|`np.concatenate((a1, a2, ...), axis)`|`axis=0`增加行；`axis=1`增加列；`axis=None`获得flat|
|`np.vstack(tup)`|增加行，入参为tuple，例如`(a1, a2, ...)`；**v**ertical方向|
|`np.hstack(tup)`|增加列；**h**orizontal方向|
|`np.r_[ar1, ar2]`|增加行，**r**ow|
|`np.c_[ar1, ar2]`|增加列，**c**olumn|
|`np.dstack(tup)`|**会拓展维度**，增加维度（深度），**d**epth|
|`np.stack(arrays, axis)`|**会拓展维度**，在新增维度上堆叠。`axis`指定新增哪个维度，`axis=0`新增第一个维度；`axis=-1`新增最后一个维度（dstack）|
||或者用`np.expand_dims`拓展维度`axis=0`后，对行进行拼接，然后`transpose`|

**demo**

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

### 增删行列

|方法|说明|
|---|---|
|`np.append(arr, values, axis=None)`|末尾添加，`axis`可指定添加行、列，shape需保证对齐|
|`np.insert(arr, obj, values, axis)`|在指定索引标号`obj`之前，沿`axis`指定轴，向数组中插入值`values`，shape需保证对齐|
|`np.delete(arr, obj, axis)`|在指定索引标号`obj`，沿`axis`指定轴，删除数组中（某行、列）数据|
|`np.unique(arr, return_index=False, return_inverse=False, return_counts=False)`|去除数组中的重复元素|

### 计算函数

|方法|说明|
|---|---|
|`np.around(a, decimals)`|四舍五入，`decimals`指定小数点后位数；负值表示round到十位、百位...|
|`np.floor(a)`|取地板|
|`np.ceil(a)`|取天花板|
||
|`np.reciprocal(a)`|取倒数|
|`np.power(a, b)`|幂|
|`np.mod(a)`|取余|
||
|`np.amin(a)` `np.nanmin(a)`|ignoring any NaNs|
|`np.amax(a)` `np.nanmax(a)`||
|`np.sum(a)` `np.nansum(a)`||
|`np.percentile(a, q)` `np.nanpercentile(a, q)`|q在0~100之间|
|`np.quantile(a, q)` `np.nanquantile(a, q)`|q在0~1之间|
|`np.median(a)` `np.nanmedian(a)`||
|`np.mean(a)` `np.nanmean(a)`||
|`np.std(a)` `np.nanstd(a)`||
|`np.var(a)` `np.nanvar(a)`||
||
|`np.dot(a, b)`|矩阵乘法，得到矩阵|
|`np.inner(a)`|向量内积（对位相乘后相加），得到一个数|

### 排序与索引查询

|方法|说明|
|---|---|
|`np.sort(a)`||
|`np.argsort(a)`|从小到大的排序后数组，对应a的索引值|
|`np.argmax(a)`|a中最大元素的索引|
|`np.argmin(a)`|a中最小元素的索引|
|`np.nonzero(a)`|a中非0元素的索引|



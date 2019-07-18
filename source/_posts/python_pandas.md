title: Python Pandas
date: 2019/07/17
categories:
- Program Development
tags:
- Python
---

## Series

Series is a one-dimensional labeled array capable of holding any data type.

#### 创建：`pd.Series(data, index)`

- 指定的index，需保证长度与data对应
- 但若data为`dict`，index长度可不与data对应。在keys内则只提取对应项；在包含keys外的项则value用NaN。
  - key对应index

```python
import numpy as np
import pandas as pd

# list
s1 = pd.Series(range(5))
'''
0    0
1    1
2    2
3    3
4    4
dtype: int64
'''
# 1darray
s2 = pd.Series(np.arange(5, dtype=np.uint64), index=['a', 'b', 'c', 'd', 'e'])
'''
a    0
b    1
c    2
d    3
e    4
dtype: uint64
'''
# dict
s3 = pd.Series({'a':'0', 'b':'1', 'c':'2', 'd':'3', 'e':'4'}, index=['a', 'b', 'c', 'xx'])
'''
a       0
b       1
c       2
xx    NaN
dtype: object
'''
```

#### 取值、切片、向量化计算等操作，与ndarray相同

#### 直接转成ndarray：`Series.to_numpy()`


## DataFrame

DataFrame is a 2-dimensional labeled data structure with columns of potentially different types.

#### 创建：`pd.DataFrame(data, index, columns)`

- 指定的index、columns，需保证长度与data对应
- 但若data为`dict`，index、columns长度可不与data对应。在keys内则只提取对应项；在包含keys外的项则value用NaN。
  - key对应column，当value是个Series，其index对应index

```python
# 2darray
d1 = np.arange(12).reshape(3, 4)
df1 = pd.DataFrame(d1, index=[997, 998, 999], columns=['A', 'B', 'C', 'D'])
print(df1)
'''
     A  B   C   D
997  0  1   2   3
998  4  5   6   7
999  8  9  10  11
'''

# dict of list
d2 = {'one': [1., 2., 3., 4.], 'two': [4., 3., 2., 1.]}
df2 = pd.DataFrame(d2, columns=['one'])
print(df2)
'''
   one
0  1.0
1  2.0
2  3.0
3  4.0
'''

# dict of Series
d3 = {'one': pd.Series([1., 2., 3., 4.]), 'two': pd.Series([4., 3., 2., 1.])}
df3 = pd.DataFrame(d3, index=[0, 2, 4], columns=['two', 'three'])
print(df3)
'''
   two three
0  4.0   NaN
2  2.0   NaN
4  NaN   NaN
'''
```

#### 修改

操作DataFrame与操作dict类似

- 获取列：
  - 指定列名称选择
    - 单列：`df1['A']`
    - 多列：`df1[['A', 'B']]`
- 获取行：
  - 连续多行选择，使用切片：`df1[0:1]`
  - 指定索引名称选择：
    - 单行：`df1.loc[997]`
    - 连续行：`df1.loc[997:999]`（**本方法较特殊，会包含'999'行**）
    - 跳行：`df1.loc[[997, 999]]`
  - 指定索引位置选择：
    - 单行：`df1.iloc[0]`
    - 连续行：`df1.iloc[0:2]`
    - 跳行：`df1.iloc[[0, 2]]`
- 删除列：`del df1['A']`
- 新增列：`df1['K'] = 'k'`（注意k会被广播）
- 去除NaN：`df1.dropna()`
- 替换NaN：`df1.fillna()`

#### 查看

- 查看头，默认5行：`df1.head(1)`
- 查看头，默认5行：`df1.tail(1)`
- 查看index：`df1.index`
- 查看columns：`df1.columns`
- 查看各列dtype：`df1.dtypes`
- 查看基本统计信息：`df1.describe()`
- 按值排序：`df1.sort_values(by='K')`

#### apply

- `df1.apply(lambda x: x.max() - x.min())`

#### 拼接

- 连接多个DataFrame：`pd.concat(df_list)`
- append行：`df1.append(s, ignore_index=True)`
- join两个DataFrame：`pd.merge(left, right, on='key')`

#### groupby

- 例如：`df1.groupby('A').sum()`、`df1.groupby('A').size()`
By "group by" we are referring to a process involving one or more of the following steps:
  - Splitting the data into groups based on some criteria
  - Applying a function to each group independently
  - Combining the results into a data structure

#### 时间索引

- 生成时间索引：`pd.date_range('2019-01-01', periods=72, freq='H')`
- 转为pandas时间格式：`pd.to_datetime(col, format='%d.%m.%Y')`
  - 设为index：`df1.set_index(col, inplace=True)`，inplace原地转换DataFrame，而不会新建

#### 数据存取

- `pd.read_csv('foo.csv')`、`df.to_csv('foo.csv')`






title: Python Pandas
date: 2019/07/17
categories:
- Program Development
tags:
- Python
---

## Series

Series is a one-dimensional labeled array capable of holding any data type.

|方法|说明|
|---|---|
|`pd.Series(data, index)`|创建，可由`list`、`np 1darray`、`dict`等创建|
||指定的`index`，需保证长度与`data`的长度对应。由`dict`创建，则可以不对应。`index`在`keys`值中设置，`keys`以外的，`value`为`NaN`。|
|`Series.to_numpy()`|直接转成`ndarray`|

索引、切片、计算等操作，与`ndarray`相同

```python
import numpy as np
import pandas as pd

# 由list创建
s1 = pd.Series(range(5))
'''
0    0
1    1
2    2
3    3
4    4
dtype: int64
'''
# 由1darray创建
s2 = pd.Series(np.arange(5, dtype=np.uint64), index=['a', 'b', 'c', 'd', 'e'])
'''
a    0
b    1
c    2
d    3
e    4
dtype: uint64
'''
# 由dict创建
s3 = pd.Series({'a':'0', 'b':'1', 'c':'2', 'd':'3', 'e':'4'}, index=['a', 'b', 'c', 'xx'])
'''
a       0
b       1
c       2
xx    NaN
dtype: object
'''
```

## DataFrame

DataFrame is a 2-dimensional labeled data structure with columns of potentially different types.

### 创建

|方法|说明|
|---|---|
|`pd.DataFrame(data, index, columns)`|创建，可由`np 2darray`、`dict`等创建|
||指定的`index`、`columns`，需保证长度与`data`的`shape`对应。由`dict`创建，则可以不对应。`columns`在`keys`值中设置（`dict`的`values`是序列，能生成`index`），`keys`以外的，`value`为`NaN`。|

```python
# 由2darray创建
d1 = np.arange(12).reshape(3, 4)
df1 = pd.DataFrame(d1, index=[997, 998, 999], columns=['A', 'B', 'C', 'D'])
print(df1)
'''
     A  B   C   D
997  0  1   2   3
998  4  5   6   7
999  8  9  10  11
'''

# 由dict of list创建
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

# 由dict of Series创建
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

### 存取

|方法|说明|
|---|---|
|`pd.read_csv('foo.csv')`||
|`df.to_csv('foo.csv')`||

**多文件合并demo**

```python
def load_data_from_csv(obj):
    if isinstance(obj, list) and len(obj) > 0:
        df_list = []
        for p in obj:
            df_list.append(pd.read_csv(p, header=0))
        all_data = pd.concat(df_list)
    else:
        all_data = pd.read_csv(obj, header=0)

    return all_data
```

### 拼接

|方法|说明|
|---|---|
|`pd.concat(df_list)`|增加行，纵向连接多个DataFrame|
|`df1.append(s, ignore_index=True)`|append行|
|`pd.merge(left, right, how='inner', on=None)`|增加列，join两个DataFrame，`how`指定join方式（左右内外），`on`指定key|

### 查看

|方法|说明|
|---|---|
|`df1.head(1)`|查看头，默认5行|
|`df1.tail(1)`|查看尾，默认5行|
|`df1.index`|查看index|
|`df1.columns`|查看columns|
||按列名称，选择列`df1[df1.columns[bool_list]]`|
|`df1.dtypes`|查看各列dtype|
|`df1.info()`|查看非空值数量等信息|
|`df1.describe()`|查看基本统计量等信息|

```python
# 数值类型
df1.describe().T.assign(missing_pct=df1.apply(lambda x: (len(x) - x.count()) / len(x)))
# 非数值类型
df1.select_dtypes(include=['object']).describe().T.assign(missing_pct=df1.apply(lambda x: (len(x) - x.count()) / len(x)))
```

### 获取与修改

|方法|说明|
|---|---|
|`df1['A']`|获取列，根据**列名**获取单列|
|`df1[['A', 'B']]`|获取列，根据**列名**list获取多列|
|||
|`df1[0:1]`|获取行，使用**切片**直接获取连续多行|
|`df1.loc[997]`|获取行，根据**索引名称**获取单行（索引名称有可能非数字）|
|`df1.loc[997:999]`|获取行，根据**索引名称**获取连续行（**本方法较特殊，会包含'999'行**）|
|`df1.loc[[997, 999]]`|获取行，根据**索引名称**获取跳行|
|`df1.iloc[0]`|获取行，根据**行位置**获取单行|
|`df1.iloc[0:2]`|获取行，根据**行位置**获取连续行|
|`df1.iloc[[0, 2]]`|获取行，根据**行位置**获取跳行|
|||
|`df1[bool_list]`|通过生成与行数同len的布尔Series，来**选择行**|
|`df1[df1['col1'] == 10]`||
|`df1[df1['col1'] != 10]`||
|`df1[df1['col1'] > 10]`||
|`df1[df1['col1'].isnull()]`||
|`df1[df1['col1'].isin([10, 20])]`|
|`df1[df1['col1'] == 10 & df1['col2'].isin([10, 20])]`||
|||
|`df[df.columns[bool_list]]`|通过生成与列数同len的布尔Series，来选择列名，进而选择列|
|`df1[df1.columns[df1.isnull().all(axis=0)]]`||
|`df[:, bool_list]`|通过生成与列数同len的布尔Series，来**选择列**||
|`df1.iloc[:, df1.isnull().all(axis=0).values]`||
|`df1.iloc[:, [df1[col].isnull().all() for col in df1.columns]]`||
|||



- 删除列：`del df1['A']`
- 新增列：`df1['K'] = 'k'`（注意k会被广播）
- `df1.assign()`
- 去除NaN：`df1.dropna()`
  - `axis : {0 or ‘index’, 1 or ‘columns’}, default 0`**此处有坑，axis操作预期与其他处不同！**
    - 0, or ‘index’ : Drop rows which contain missing values.
    - 1, or ‘columns’ : Drop columns which contain missing value.
  - `how : {‘any’, ‘all’}, default ‘any’`
    - ‘any’ : If any NA values are present, drop that row or column.
    - ‘all’ : If all values are NA, drop that row or column.
  - `thresh : int, optional`
    - ‘any’、‘all’的折中，达到‘thresh’个NaN再去除
  - `subset : array-like, optional`
    - Labels along other axis to consider, e.g. if you are dropping rows these would be a list of columns to include.
    - 指定考察哪些（字段）
  - `inplace : bool, default False`
    - If True, do operation inplace and return None.
- 替换NaN：`df1.fillna()`
- 按值排序：`df1.sort_values(by='K')`


#### apply

- `df1.apply(lambda x: x.max() - x.min())`




#### 汇聚

groupby
- 例如：`df1.groupby('A').sum()`、`df1.groupby('A').size()`
By "group by" we are referring to a process involving one or more of the following steps:
  - Splitting the data into groups based on some criteria
  - Applying a function to each group independently
  - Combining the results into a data structure

透视表


#### 时间索引

- 生成时间索引：`pd.date_range('2019-01-01', periods=72, freq='H')`
- 转为pandas时间格式：`pd.to_datetime(col, format='%d.%m.%Y')`
  - 设为index：`df1.set_index(col, inplace=True)`，inplace原地转换DataFrame，而不会新建





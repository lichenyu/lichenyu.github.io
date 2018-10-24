title: SQL Cheatsheet
date: 2018/10/24
categories:
- Program Development
tags:
- SQL
---


## 1. 动词 ##

|||
|---|---|
|数据操作|`SELECT`  `UPDATE`  `INSERT`  `DELETE`|
|数据定义|`CREATE`  `ALTER`  `DROP`|
|数据控制|`GRANT`  `REVOKE`|


## 2. 语句形式 ##

```sql
SELECT [DISTINCT/是否去重] <目标列表达式> [, <目标列表达式>...]
FROM <表名或视图名> [, <表名或视图名>...]
[WHERE <条件表达式>]
[GROUP BY <列名>]
[ORDER BY <列名> [ASC|DESC] [, ...]];

```

- 大小写不敏感
- `*`代表选取所有列
- `GROUP BY`指定“聚集函数”（对于某字段，多行输入，单值输出，例如`COUNT`、`SUM`、`AVG`、`MAX`、`MIN`）的作用对象。即：未使用，聚集函数将作用于整个查询结果；使用后，聚集函数将分别作用于各个组（每个组所有行单独计算）。
- 聚集函数无法与`WHERE`一起使用时，使用`HAVING`。

```sql
SELECT Customer, SUM(OrderPrice) FROM Orders
WHERE Customer='Bush' OR Customer='Adams'
GROUP BY Customer
HAVING SUM(OrderPrice)>1500
```


## 3. 运算符 ##

|||
|---|---|
|比较|`=`  `>`  `<`  `>=`  `<=`  `<>`  `!=`|
|字符匹配|`LIKE`|
|确定集合|`IN`|
|确定范围|`BETWEEN`|
|判断空值|`IS NULL`|
|逻辑运算|`AND`  `OR`  `NOT`|

**WHERE中条件运算符**，即`WHERE 列名 XXX运算符`：

字符匹配：`LIKE 通配符`

|||
|---|---|
|`%`|替代一个或多个字符|
|`_`|仅替代一个字符|
|`[charlist]`|字符列中的任何单一字符（配合`%`、`_`使用）|
|`[^charlist]`|不在字符列中的任何单一字符（配合`%`、`_`使用）|

所属集合：`IN (value1, value2, ...)`

范围选取：`BETWEEN ... AND ...`


## 4. 别名 ##

使用表名称别名：

```sql
SELECT po.OrderID, p.LastName, p.FirstName
FROM Persons AS p, Product_Orders AS po
WHERE p.LastName='Adams' AND p.FirstName='John'
```

使用一个列名别名（会体现在查询结果上）：

```sql
SELECT LastName AS Family, FirstName AS Name
FROM Persons
```


## 5. 多表操作 ##

### 5.1. 引用多个表 ###

```sql
SELECT 字段名 FROM 表1, 表2
WHERE 表1.字段 = 表2.字段 AND 其它查询条件
```

- 两个表相连接，仅取出符合条件的匹配数据。


### 5.2. INNER JOIN ###

```sql
SELECT * FROM A
INNER JOIN B
ON A.aID = B.bID
```

- 与“引用多个表”功能相同


### 5.3. LEFT JOIN ###

```sql
SELECT * FROM A
LEFT JOIN B
ON A.aID = B.bID
```

- 首先取出A表中所有数据，然后迭代A.aID来检查B表各项的bID，若匹配则扩展B表数据，否则补NULL。换句话说，左表A的记录将会全部显示出来，而右表B只会显示符合匹配条件的记录。


### 5.4. RIGHT JOIN ###

```sql
SELECT * FROM A
RIGHT JOIN B
ON A.aID = B.bID
```

- 首先取出B表中所有数据，然后迭代B.bID来检查A表各项的aID，若匹配则扩展A表数据，否则补NULL。


### 5.5. FULL JOIN ###

```sql
SELECT * FROM A
FULL JOIN B
ON A.aID = B.bID
```

- 左连接后，将右表的剩余项NULL拓展后，加入结果表。


### 5.6. UNION ###

```sql
SELECT column_name(s) FROM table_name1
UNION [ALL]
SELECT column_name(s) FROM table_name2
```

- 将多表（的SELECT内容）进行**行**连接
- UNION默认选取不同的值。如果允许重复的值，使用UNION ALL。


### 5.7. SELECT ... INTO ... ###

```sql
SELECT column_name(s)
INTO new_table_name [IN external_database]
FROM old_tablename
```

- SELECT INTO语句从一个表中选取数据，然后把数据**行**插入另一个表中。常用于创建表的备份复件或者用于对记录进行存档。


## 6. 行列操作 ##

### 6.1. 插入行 INSERT INTO ... VALUES ...###

```sql
INSERT INTO 表名 VALUES (值1, 值2, ...)
INSERT INTO 表名 (列1, 列2, ...) VALUES (值1, 值2, ...)
```


### 6.2. 更新行 UPDATE ... SET ... ###

```sql
UPDATE 表名 SET 列名称=新值 [, ...] WHERE <条件表达式>
```


### 6.3. 删除行 DELETE ###

```sql
DELETE FROM 表名 WHERE <条件表达式>
```


### 6.4. 添加列 ALTER ... ADD ... ###

```sql
ALTER TABLE table_name
ADD column_name datatype
```


### 6.5. 更新列 ALTER ... ALTER ... ###

```sql
ALTER TABLE table_name
ALTER COLUMN column_name datatype
```


### 6.6. 删除列 ALTER ... DROP ... ###

```sql
ALTER TABLE table_name 
DROP COLUMN column_name
```


## 7. 聚集函数 ##

### 7.1. AVG ###

```sql
SELECT AVG(column_name) FROM table_name
```

- AVG()函数返回数值列的平均值。NULL 值不包括在计算中。


### 7.2. COUNT ###

- COUNT()函数返回匹配指定条件的行数。
- COUNT(column_name)：函数返回指定列的值的数目（NULL 不计入）
- COUNT(DISTINCT column_name)：函数返回指定列的不同值的数目
- COUNT(*)：函数返回表中的记录数


### 7.3. SUM ###

```sql
SELECT SUM(column_name) FROM table_name
```

- SUM()函数返回数值列的总数（总额）。


### 7.4. FIRST、LAST ###

```sql
SELECT FIRST(column_name) FROM table_name
SELECT LAST(column_name) FROM table_name
```

- FIRST()函数返回指定的字段中第一个记录的值。
- LAST()函数返回指定的字段中最后一个记录的值。


### 7.5. MAX、MIN ###

```sql
SELECT MAX(column_name) FROM table_name
SELECT MIN(column_name) FROM table_name
```

- MAX()函数返回一列中的最大值。NULL 值不包括在计算中。
- MIN()函数返回一列中的最小值。NULL 值不包括在计算中。


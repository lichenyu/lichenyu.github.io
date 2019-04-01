title: Python - list、dict遍历删除
date: 2019/04/01
categories:
- Program Development
tags:
- Python
---


## list遍历删除 ##

反序遍历list，使得idx不会超过更改后的list下标范围

```python
mylist = [1, 2, 3, 4, 5]
for i in range(len(mylist) - 1, -1, -1):
    if mylist[i] == 3:
        del mylist[i]
print(mylist)

```


## dict遍历删除 ##

将key构建成list，按此list查询遍历dict，可直接del

```python
mydic = {"k1": "v1", "k2": "v2", "k3": "v3", "k4": "v4", "k5": "v5"}
for k in list(mydic):
    if mydic[k] == "v3":
        del mydic[k]
print(mydic)

```
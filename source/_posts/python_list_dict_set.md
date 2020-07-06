title: Python - list、dict、set常用操作
date: 2020/07/06
categories:
- Program Development
tags:
- Python
---


```python
# coding=utf-8


def list_demo():
    print('创建')
    list_0 = []
    list_1 = [0, 1, 2, 3, 4]
    list_2 = [x for x in range(10)]
    print('list_0 = {}'.format(list_0))
    print('list_1 = {}'.format(list_1))
    print('list_2 = {}'.format(list_2))

    print('按下标索引')
    print('list_1[0] = {}'.format(list_1[0]))

    print('按下标、切片修改')
    list_1[0] = 999
    list_2[0:3] = [6, 66, 666]
    print('list_1 = {}'.format(list_1))
    print('list_2 = {}'.format(list_2))

    print('增加元素、拓展列表')
    list_1.append(0)
    list_2 += [0]
    print('list_1 = {}'.format(list_1))
    print('list_2 = {}'.format(list_2))

    print('按值删除、按下标删除')
    list_1.remove(999)
    del list_2[len(list_2)-1]
    print('list_1 = {}'.format(list_1))
    print('list_2 = {}'.format(list_2))

    print('遍历并按条件删除')
    for item in list_1[:]:
        if item == 4:
            del item
    for i in range(len(list_1)-1, -1, -1):
        if list_2[i] == 4:
            del list_1[i]
    print('list_1 = {}'.format(list_1))
    print('list_2 = {}'.format(list_2))

    print('排序')
    new_list = sorted(list_1, key=lambda x: -x)
    print(new_list)


def dict_demo():
    print('创建')
    dict_0 = {}
    dict_1 = {0: 'zero', 1: 'one', 2: 'two', 3: 'three', 4: 'four'}
    dict_2 = {x: x**2 for x in range(10)}
    print('list_0 = {}'.format(dict_0))
    print('list_1 = {}'.format(dict_1))
    print('list_2 = {}'.format(dict_2))

    print('按下标索引')
    print('dict_1[0] = {}'.format(dict_1[0]))
    print('dict_1[999] = {}'.format(dict_1.get(999, 'default')))

    print('按下标、切片修改')
    dict_1[0] = 999
    print('dict_1 = {}'.format(dict_1))

    print('增加元素、拓展字典')
    dict_1[-1] = 'minus one'
    dict_2.update(dict_1)
    print('dict_1 = {}'.format(dict_1))
    print('dict_2 = {}'.format(dict_2))

    print('按下标删除')
    del dict_2[-1]
    print('dict_2 = {}'.format(dict_2))

    print('遍历并按条件删除')
    for k in list(dict_2.keys()):
        if k == 2 or dict_2[k] == 'three':
            del dict_2[k]
    print('dict_2 = {}'.format(dict_2))

    print('按value排序')
    dict_2[-1] = 999
    new_list = sorted(dict_2.items(), key=lambda item: (str(item[1]), -item[0]))
    print(new_list)


def set_demo():
    print('创建')
    set_0 = set()
    set_1 = {0, 1, 2, 3, 4, 4, 4}
    set_2 = {x for x in range(10)}
    print('list_0 = {}'.format(set_0))
    print('list_1 = {}'.format(set_1))
    print('list_2 = {}'.format(set_2))

    print('增加元素、拓展字典')
    set_1.add(999)
    set_1.update(set_2)
    print('set_1 = {}'.format(set_1))

    print('按元素删除')
    set_1.remove(999)
    print('set_1 = {}'.format(set_1))


if __name__ == '__main__':
    list_demo()
    print(''.center(100, '-'))
    dict_demo()
    print(''.center(100, '-'))
    set_demo()


```
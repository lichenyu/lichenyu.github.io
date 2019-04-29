title: Python - 目录操作
date: 2019/04/29
categories:
- Program Development
tags:
- Python
---


## 目录名中包含中文 ##

```python
root_dir = "D:/工作留档/"
root_dir = unicode(root_dir, "utf-8")
if not os.path.exists(root_dir):
    print("Error: %s does not exist!" % root_dir)
    exit(-1)
```


## 文件内容中包含中文 ##

```python
info_path = os.path.join(root_dir, "info_file")
info_fd = codecs.open(info_path, "w", "utf-8")
info_fd.close()
```


## 清空目录 ##

```python
plot_dir = os.path.join(root_dir, "plot")
if os.path.exists(plot_dir):
    import shutil
    shutil.rmtree(plot_dir)
os.mkdir(plot_dir)
```


## 计数文件夹中内容 ##

```python
def count_dir(root_dir):
    cur_items = 0
    for filename in os.listdir(root_dir):
        path = os.path.join(root_dir, filename)
        if os.path.isdir(path):
            cur_items += count_dir(path)
        elif path.endswith(".data"):
            cur_items += 1
    return cur_items
```


## 处理文件夹中内容 ##

```python
def check_dir(root_dir, info_path, checked_items, total_items):
    for filename in os.listdir(root_dir):
        path = os.path.join(root_dir, filename)
        if os.path.isdir(path):
            checked_items = check_dir(path, info_path, checked_items, total_items)
        elif path.endswith(".data"):
            parse_data(path, info_path)
            checked_items += 1
            print("(%d/%d)\t%0.2f%%" % (checked_items, total_items, checked_items * 100.0 / total_items))
    return checked_items
```

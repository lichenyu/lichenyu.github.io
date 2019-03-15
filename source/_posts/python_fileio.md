title: Python - 文件IO
date: 2019/03/15
categories:
- Program Development
tags:
- Python
---


## 文件打开/关闭 ##

文件打开：`file object = open(file_name [, access_mode][, buffering])`

- `access_mode`：决定打开文件的模式。这个参数是非强制的，默认文件访问模式为只读(`r`)。
- `buffering`：如果`buffering`的值被设为0，就不会有寄存。如果`buffering`的值取1，访问文件时会寄存行。如果将`buffering`的值设为大于1的整数，表明了这就是的寄存区的缓冲大小。如果取负值，寄存区的缓冲大小则为系统默认。

打开方式：

- `b`：二进制模式。
- `+`：打开一个文件进行更新(可读可写)。

|模式|r|r+|w|w+|a|a+|
|---|---|---|---|---|---|---|
|读|+|+||+||+|
|写||+|+|+|+|+|
|创建|||+|+|+|+|
|覆盖|||+|+|||
|指针在开始|+|+|+|+|||
|指针在结尾|||||+|+|

文件关闭：`fileObject.close()`


## 读文件 ##

读取指定字节：`fileObject.read([count])`
读取一行：`fileObject.readline()`
读取所有行：`fileObject.readlines()`

高性能读逐行：
```python
with open("filename") as fd:
    for line in fd:
        do_things(line)
```


## 写文件 ##

写文件：`fileObject.write(string)`

- `write()`方法可将任何字符串写入一个打开的文件。需要重点注意的是，Python字符串可以是二进制数据，而不是仅仅是文字。
- `write()`方法不会在字符串的结尾添加换行符`\n`。

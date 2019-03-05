title: Python - logging日志模块
date: 2019/03/05
categories:
- Program Development
tags:
- Python
---


## 日志配置直接写在代码中 ##

### 日志模块 ###

```python
# -*- coding: UTF-8 -*-

import logging
import sys
import os

"""
Python中，类实例化的过程是先执行Singleton.__new__生成instance，
然后执行instance.__init__进行初始化的。
所以通过重写__new__可以达到所有调用Singleton()的地方都返回同一个对象。
"""
class Singleton(object):
    _instance = None
    def __new__(class_, *args, **kwargs):
        if not isinstance(class_._instance, class_):
            class_._instance = object.__new__(class_, *args, **kwargs)
        return class_._instance

"""
单例模式，适应程序中有多个入口，只进行一次配置。
"""
class Logger(Singleton):
    def __init__(self):
        import detector_info as di
        self.logger = logging.getLogger(di.name)
        # 进入handler的日志级别
        self.logger.setLevel(logging.DEBUG)

        stream_handler = logging.StreamHandler(sys.stdout)
        stream_handler.setLevel(logging.DEBUG)
        formatter = logging.Formatter(fmt="%(message)s", datefmt="%Y/%m/%d %H:%M:%S")
        stream_handler.setFormatter(formatter)
        self.logger.addHandler(stream_handler)

        import config.global_config as gc
        file_handler = logging.FileHandler(os.path.join(gc.run_dir, "log", "argogo.log"))
        file_handler.setLevel(logging.DEBUG)
        formatter = logging.Formatter(fmt="%(asctime)s | %(levelname)s | %(message)s", datefmt="%Y/%m/%d %H:%M:%S")
        file_handler.setFormatter(formatter)
        self.logger.addHandler(file_handler)

```

### 初始化 ###

在主模块中

```python
import log.detector_log
log = log.detector_log.Logger().logger
```

### 引入 ###

其他模块中

```python
import detector_info as di
log = logging.getLogger(di.name)
```

## 日志配置写在配置文件中 ##

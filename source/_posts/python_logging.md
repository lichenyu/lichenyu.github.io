title: Python - logging日志模块
date: 2019/03/05
categories:
- Program Development
tags:
- Python
---


## 1. 日志配置直接写在代码中 ##

### 日志模块 ###

```python
# -*- coding: UTF-8 -*-

import logging
import logging.config
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
单例模式，以适配场景：程序中有多个模块有main入口，都会import本模块；而当这些模块相互import时，logger只应进行一次配置。
"""
class Logger(Singleton):
    def __init__(self):
        import detector_info as di
        self.logger = logging.getLogger(di.name)
        # 进入handler的日志级别
        self.logger.setLevel(logging.DEBUG)

        stream_handler = logging.StreamHandler(sys.stdout)
        stream_handler.setLevel(logging.DEBUG)
        formatter = logging.Formatter("%(message)s")
        stream_handler.setFormatter(formatter)
        self.logger.addHandler(stream_handler)

        log_dir = os.path.dirname(os.path.realpath(__file__))
        log_file_path = os.path.join(log_dir, "argogo.log")
        file_handler = logging.FileHandler(log_file_path)
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

## 2. 日志配置写在配置文件中 ##

### 配置文件 ###

```plain
#--------------------------------------------------

[loggers]
keys = root, dlog

[logger_root]
level = DEBUG
handlers = console_handler

[logger_dlog]
level = DEBUG
handlers = console_handler, file_handler
# to avoid duplicated log with root logger
propagate = 0
qualname = %(logger_name)s

#--------------------------------------------------

[handlers]
keys = console_handler, file_handler

[handler_console_handler]
class = StreamHandler
level = DEBUG
formatter = simple_format
args = (sys.stdout,)

[handler_file_handler]
class = FileHandler
level = DEBUG
formatter = detailed_format
args = (r"%(log_file_path)s",)

#--------------------------------------------------

[formatters]
keys = detailed_format, simple_format

[formatter_detailed_format]
format = %(asctime)s | %(levelname)s | %(message)s
datefmt = %Y/%m/%d %H:%M:%S

[formatter_simple_format]
format = %(message)s

#--------------------------------------------------
```

使用标准config file格式，注意`=`周围的空格会被忽略。

### 日志模块 ###

```python
# -*- coding: UTF-8 -*-

import logging
import logging.config
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
单例模式，以适配场景：程序中有多个模块有main入口，都会import本模块；而当这些模块相互import时，logger只应进行一次配置。
"""
class Logger(Singleton):
    def __init__(self):
        log_dir = os.path.dirname(__file__)
        log_config_path = os.path.join(log_dir, "log.config")
        log_file_path = os.path.join(log_dir, "argogo.log")
        import detector_info as di
        logging.config.fileConfig(log_config_path, defaults={"logger_name": di.name, "log_file_path": log_file_path})
        self.logger = logging.getLogger(di.name)

```

### 初始化及引入方式不变 ###

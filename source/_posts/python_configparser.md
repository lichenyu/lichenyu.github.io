title: Python - ConfigParser配置模块
date: 2019/03/07
categories:
- Program Development
tags:
- Python
---


## 配置读取模块 ##

```python
import ConfigParser
import os


class Config:
    def __init__(self, config_path=os.path.join(os.path.dirname(os.path.realpath(__file__)), "argogo.config")):
        self.config_path = config_path

    def read_config(self):
        config = ConfigParser.ConfigParser()
        config.read(self.config_path)
        return config

    def get_value(self, section, key):
        config = ConfigParser.ConfigParser()
        config.read(self.config_path)
        return config.get(section, key)

```

## 配置文件 ##

```plain
# config file

[section1]
key1 = value1

[section2]
key2 = value 2

```

使用标准config file格式，注意`=`周围的空格会被忽略。

## 使用 ##

```python
import config.config_parser
conf = config.config_parser.Config().read_config()

import config.global_config as gc
gc.key1 = conf.get("section1", "key1")
gc.key2 = config.config_parser.Config().get_value("section2", "key2")
log.info("key1 = " + gc.key1)
log.info("key2 = " + gc.key2)

```

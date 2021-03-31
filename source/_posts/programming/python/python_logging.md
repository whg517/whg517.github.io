---
title: Python Logging 使用实践
author: Kevin
date: 2021-03-31 16:20:00
updated: 2021-03-31 16:20:00
tags:
- Python
- Logging
categories: 开发
---

日志是开发过程中一个重要的环节，贯穿开发到部署。良好的日志习惯能提高开发调试效率，而且可以在程序出问题时，快速精准定位问题所在，及时修复
BUG。

<!-- more -->

## 一、Python 日志基础

相关文章：

[日志 HOWTO](https://docs.python.org/zh-cn/3/howto/logging.html#logging-basic-tutorial)

[Python 的日志记录工具](https://docs.python.org/zh-cn/3/library/logging.html)

## 二、Python 日志实践

下面开始实践，基础内容不在赘述，其中要点会引用相关文档

### 1. 简单使用

本情况适用于临时需要使用日志系统，或者简单使用的场景。

```python
import logging

logging.warning('I love you ~')
logging.info('I love you too ~~~')  # 这条日志是永远不会打印出来的，它永远不会回爱你。
logging.error('I hate you !')  # 这条日志会打印出来！
```

输入如下：

```plain text
WARNING:root:I love you ~
ERROR:root:I hate you !
```

根据文档 [日志级别](https://docs.python.org/zh-cn/3/howto/logging.html#logging-levels) 中描述，日志分为多个等级。
从上述示例中可以看到日志的默认等级是 `WARNING` ，低于此等级的日志默认情况下是不会输出的。

在实际开发中，推荐使用日志的方式来输出程序信息，而不是使用 `print` 。当你在开发的时候先使用了 `print` 输出信息，在后期需要
改用日志的时候，就会改动很多代码。而且使用日志后，可以很方便的调整日志等级，过滤你想看到的内容。

#### 1.1 自定义日志配置

```python
import logging

logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    datefmt='%Y-%m-%dT%H:%M:%S.%s+0800',
)

logging.debug('I love you~')
logging.info('I love you too ~')
```

输出如下：

```plain text
2021-03-31T16:30:14.1617179414+0800 - root - DEBUG - I love you~
2021-03-31T16:30:14.1617179414+0800 - root - INFO - I love you too ~
```

上述示例为日志配置了日志级别，日志格式和时间格式。其中使用了 `DEBUG` 级别显示日志，使用日志格式字符串显示格式化的日志内容，
而且在显示时间上也做了格式化，符合 [ISO8601](https://en.wikipedia.org/wiki/ISO_8601) 。

日志格式化参数可以参考 [LogRecord 属性](https://docs.python.org/zh-cn/3/library/logging.html#logrecord-attributes) 中
包含的属性名称。如果需要显示更详细的日志信息，可以自行添加格式化字段，如 `filename` 、 `process` 、 `thread` 等。

推荐的几个日志格式：

**简单信息:**

`%(asctime)s - %(name)s - %(levelname)s - %(message)s`

输出示例：

```plain text
2021-03-31T17:05:29.1617181529+0800 - root - DEBUG - I love you~
2021-03-31T17:05:29.1617181529+0800 - root - INFO - I love you too ~
```

**一般信息:**

`%(asctime)s - %(name)s - %(levelname)s - %(module)s - %(process)d %(thread)d - %(message)s`

输出示例：

```plain text
2021-03-31T17:05:42.1617181542+0800 - root - DEBUG - simple_logger - 19146 140370774697792 - I love you~
2021-03-31T17:05:42.1617181542+0800 - root - INFO - simple_logger - 19146 140370774697792 - I love you too ~
```

**详细信息:**

`%(asctime)s - %(name)s - %(levelname)s - %(module)s - %(process)d %(thread)d - %(pathname)s:%(lineno)d %(message)s`

输出示例：

```plain text
2021-03-31T17:05:54.1617181554+0800 - root - DEBUG - simple_logger - 19343 139933362054976 - /foo/simple_logger.py:13 I love you~
2021-03-31T17:05:54.1617181554+0800 - root - INFO - simple_logger - 19343 139933362054976 - /foo/simple_logger.py:14 I love you too ~
```

#### 1.2 增加日志对象

```python
import logging

formatter = logging.Formatter(
    fmt='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    datefmt='%Y-%m-%dT%H:%M:%S.%s+0800',
)

# 创建控制台 handler
console_handler = logging.StreamHandler()
console_handler.setFormatter(formatter)
console_handler.setLevel(logging.DEBUG)

# 创建文件 handler
file_handler = logging.FileHandler(filename='foo.log')
file_handler.setFormatter(formatter)
file_handler.setLevel(logging.DEBUG)

# 创建 foo logger
foo_logger = logging.getLogger('foo')
foo_logger.addHandler(console_handler)
foo_logger.addHandler(file_handler)
foo_logger.setLevel(logging.INFO)

# 创建 bar logger
bar_logger = logging.getLogger('foo')
bar_logger.addHandler(console_handler)
bar_logger.setLevel(logging.DEBUG)

foo_logger.debug('This debug msg')
foo_logger.info('This info msg')

bar_logger.debug('This debug msg')
bar_logger.info(f'This info msg')

```

输出如下：

```plain text
2021-03-31T16:40:18.1617180018+0800 - foo - DEBUG - This debug msg
2021-03-31T16:40:18.1617180018+0800 - foo - INFO - This info msg
2021-03-31T16:40:18.1617180018+0800 - foo - DEBUG - This debug msg
2021-03-31T16:40:18.1617180018+0800 - foo - INFO - This info msg
```

### 2. 日志配置文件

[配置字典的 Schema](https://docs.python.org/zh-cn/3/library/logging.config.html#configuration-dictionary-schema) 配置字段都在
 [Schema 详情](https://docs.python.org/zh-cn/3/library/logging.config.html#dictionary-schema-details) 中。

在 [配置日志记录](https://docs.python.org/zh-cn/3/howto/logging.html#configuring-logging) 中讲了配置日志的几种方式。

下面通过几个 Example 配置说明

#### 2.1 Dict

logging.py

```python
import logging
from logging.config import dictConfig

LOGGING_CONFIG = {
    'version': 1,
    'disable_existing_loggers': False,
    'filters': {
        'require_debug_false': {
            # 这里的小括号是使用了日志自定义日志对象。参考 https://docs.python.org/zh-cn/3/library/logging.config.html#user-defined-objects
            '()': 'django.utils.log.RequireDebugFalse'
        },
        'require_debug_true': {
            '()': 'django.utils.log.RequireDebugTrue'
        },
    },

    'formatters': {
        'detailed': {
            'class': 'logging.Formatter',
            'format': '%(asctime)s %(name)-15s %(levelname)-8s %(processName)-10s %(message)s'
        }
    },
    'handlers': {
        'console_handle': {
            'class': 'logging.StreamHandler',
            'level': 'INFO',
            # 这里的 stream 是 StreamHandler 的参数。该参数也可以是 ext://sys.stderr
            # 参考 https://docs.python.org/zh-cn/3/library/logging.config.html#access-to-external-objects
            "stream": "ext://sys.stdout",
        },
        'access_handle': {
            'class': 'logging.FileHandler',
            'filename': 'access-file.log',
            'formatter': 'detailed',
            'mode': 'w',
        },
        'rotating_file_handle': {
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': 'rotating-file.log',
            'formatter': 'detailed',
            # 后面三个参数是 RotatingFileHandler 的参数。
            # 参考 https://docs.python.org/zh-cn/3/library/logging.config.html#dictionary-schema-details 的 handles
            'mode': 'a',
            'maxBytes': 1024 * 1024,
            'backupCount': 10,
        },
        'errors_handle': {
            'class': 'logging.FileHandler',
            'filename': 'error.log',
            'level': 'ERROR',
            'formatter': 'detailed',
            'mode': 'w',
        },
    },
    'loggers': {
        'access': {
            'handlers': ['access_handle']
        },
        'foo': {
            'handlers': ['access_handle'],
        },
        # logger 的 name 的点可以起到日志分级的作用。子级 foo.bar 日志可以传递到父级 foo ，并被捕获。
        # 所以在创建日志对象的时候推荐使用 `logging.getLogger(__name__)`。这样可以使用Python的包的层级。
        # 参考 https://docs.python.org/zh-cn/3/library/logging.html#logger-objects
        'foo.bar': {
            'handlers': ['access_handle'],
            'level': 'DEBUG'
        }
    },
    'root': {
        'level': 'DEBUG',
        'handlers': ['console_handle', 'access_handle', 'errors_handle']
    },
}

logging.config.dictConfig(LOGGING_CONFIG)
```

在 Python 中使用字典形式是常见的操作，因为这是最早支持的方式。在 `python 3.2` 引入了 yaml 的方式

#### 2.2 yaml

logging.yml

```yaml
version: 1
disable_existing_loggers: False
formatters:
  simple:
    format: '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
handlers:
  console:
    class: logging.StreamHandler
    level: DEBUG
    formatter: simple
    stream: ext://sys.stdout
loggers:
  simpleExample:
    level: DEBUG
    handlers: [console]
    propagate: no
  uvicorn.error:
    level: INFO
  uvicorn.acdess:
    level: INFO
    propagate: False
    handles: [access]
root:
  level: DEBUG
  handlers: [console]
```

使用:

先安装 yaml 解析工具。这里使用 pyyaml，其他解析库参考 [yaml](https://yaml.org/)

```bash
pip install pyyaml
```

编码：

```python
import logging
from logging.config import dictConfig
from yaml import load

try:
    from yaml import CLoader as Loader, CDumper as Dumper
except ImportError:
    from yaml import Loader, Dumper

LOGGING_CONFIG = load(open('logging.yaml'), Loader=Loader)
logging.config.dictConfig(LOGGING_CONFIG)
```

上面对于参数都已经做了说明，yaml 示例就不在作解释。

#### 2.3 ini

参考 [配置文件格式](https://docs.python.org/zh-cn/3/library/logging.config.html#configuration-file-format)

logging.ini / logging.cfg / logging.conf

```ini
[loggers]
keys=root,simpleExample

[handlers]
keys=consoleHandler

[formatters]
keys=simpleFormatter

[logger_root]
level=DEBUG
handlers=consoleHandler

[logger_simpleExample]
level=DEBUG
handlers=consoleHandler
qualname=simpleExample
propagate=0

[handler_consoleHandler]
class=StreamHandler
level=DEBUG
formatter=simpleFormatter
args=(sys.stdout,)

[formatter_simpleFormatter]
format=%(asctime)s - %(name)s - %(levelname)s - %(message)s
datefmt=
```

使用:

```python
import logging
from logging.config import fileConfig

fileConfig('logging.ini')
```

相对来说，yaml 格式在可读性上更强。但是在可操作性上，字典格式可以直接写在 python 文件中，而且可以根据变量灵活判断。

### 3. 声明式定义日志级别

```python
import logging
from logging.handlers import RotatingFileHandler

# format
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')

# Handler
rotating_file_handle = RotatingFileHandler('rotating-file.log', maxBytes=1024, backupCount=10)
rotating_file_handle.setLevel(logging.DEBUG)
rotating_file_handle.setFormatter(formatter)

stream_handler = logging.StreamHandler()
stream_handler.setLevel(logging.DEBUG)
stream_handler.setFormatter(formatter)

# loagger
logging.root.setLevel(logging.DEBUG)

foo_logger = logging.getLogger('foo')
foo_logger.setLevel(logging.DEBUG)
foo_logger.addHandler(rotating_file_handle)

foot_bar_logger = logging.getLogger('foo.bar')
foot_bar_logger.setLevel(logging.WARN)

```

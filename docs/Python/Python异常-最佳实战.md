# Python异常-最佳实战

## 前言

在Python中，异常类通常要继承Exception类。在Python中，exception和error的区别并不像Java语言那么明确。比如在Python的源码中，有如下的异常类：

```Python
class ValueError(Exception):
    pass
```

可见在python中，error和exception是混用的。



## 自定义异常

1. 定义异常

```Python
class BizException(Exception):  # 一定要继承Exception类
    pass
```

2. 抛出异常

```Python
raise Exception('故意抛出异常！')  # 手动抛出异常，通常用于替换原来的异常，因为原来的异常信息可能不容易找到异常原因
```

3. 捕获异常

```Python
# 建议采用下面的装饰器捕获异常
# 建议装饰器放在最上层函数上。
```

## 日志记录异常

1. 自定义装饰器：下面的代码单独放在文件中，文件可以命名为``record_exception_log.py``。

需要``pip install objprint``。

```Python
import logging
import os
import traceback
from logging import StreamHandler
from logging.handlers import TimedRotatingFileHandler

from objprint import objstr  # pip install objprint

filename = "logs/error/error.log"
if not os.path.exists(os.path.dirname(filename)):  # 如果目录不存在，则创建该目录
    os.makedirs(os.path.dirname(filename))
logger = logging.getLogger(__name__)  # 日志对象名称
logger.setLevel(level=logging.INFO)  # 日志级别
# 下面设置每隔一段时间生成一份新的日志文件，且日志文件的总数小于一个值
timeRotatingFilehandler = TimedRotatingFileHandler(filename, when="D", interval=1,
                                                   backupCount=60, encoding='utf-8')  # 指定日志名，计时单位，计时间隔，最多文件数量
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')  # 日志格式
timeRotatingFilehandler.setFormatter(formatter)
streamHandler = StreamHandler()
logger.addHandler(timeRotatingFilehandler)  # 一定要给logger对象加入handler
logger.addHandler(streamHandler)  # 一定要给logger对象加入handler


def record_exception_log(f):
    def wrapper(*args, **kwargs):
        try:
            return f(*args, **kwargs)
        except Exception as e:
            logger.error("======error log start======\n")
            logger.error("function name:\t")
            logger.error(f.__name__)
            logger.error("args:\t")
            logger.error(objstr(args))
            logger.error('error reason:')
            logger.error(e.__str__())
            logger.error("error logs:\t")
            logger.error(traceback.format_exc())
            logger.error("======error log end======\n")
            # 有时候我们还需要将异常往上抛，比如让全局异常装饰器捕获异常，并返回自定义错误信息
            raise e

    return wrapper
```

2. 用于某个函数的定义上面

```Python
@record_exception_log()
def func(a, b, c):
    try:
        # 代码 ...
    except Exception as e:
        raise Exception('手动抛出异常，异常详情:' + str(e))
```
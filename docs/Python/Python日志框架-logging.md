# Python日志框架-logging



## 为什么不直接使用print

1. 使用Python的logging模块可以自定义显示不同级别的日志信息。
2. logging可以自定义将日志内容输入到某个地方，而print只能输出到标准输出中。



## 关于handler

logging模块中有很多种handler类。不同的handler对象可以将日志信息输入到不同的地方。



## 基本知识

1. 建议使用logger对象打印日志。
2. 建议logger对象中添加1个或多个handler对象。
3. 常用的Handler有StreamHandler、FileHandler、TimedRotatingFileHandler。

关于StreamHandler和FileHandler：

```Python
import logging

# logger.setLevel > handler.setLevel > logging.basicConfig
logger = logging.getLogger(__name__)  # 日志对象名称
logger.setLevel(level=logging.DEBUG)  # 日志级别
handler = logging.StreamHandler()  # handler类型
# handler = logging.FileHandler("log.txt")
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')  # 日志格式
handler.setFormatter(formatter)
logger.addHandler(handler)  # 一定要给logger对象加入handler

logger.info("Start print log")
logger.debug("Do something")
logger.warning("Something maybe fail.")
logger.info("Finish")
```

关于TimedRotatingFileHandler

```Python
import logging
import time
from logging.handlers import TimedRotatingFileHandler


logger = logging.getLogger(__name__)  # 日志对象名称
logger.setLevel(level=logging.DEBUG)  # 日志级别
handler = TimedRotatingFileHandler("demo.log", when="s", interval=1, backupCount=15)  # 指定日志文件名，计时单位，计时间隔，最多文件数量
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')  # 日志格式
handler.setFormatter(formatter)
logger.addHandler(handler)  # 一定要给logger对象加入handler

for i in range(3000):
    logger.info(i)
    print(i)  # 用于debug
    time.sleep(0.3)
```



## 最佳实战

文件路径：根目录/handler/Loghandler.py

```Python
import logging
import os.path
from logging.handlers import TimedRotatingFileHandler


# 路径规范:根目录/handler/logHandler.py
CURRENT_PATH = os.path.dirname(os.path.abspath(__file__))
ROOT_PATH = os.path.join(CURRENT_PATH, os.pardir)
# 确保根路径下面有log文件夹
LOG_PATH = os.path.join(ROOT_PATH, 'log')


class LogHandler(logging.Logger):
    def __init__(self, log_file_name='log',  # 指定日志文件名称
                 min_level=logging.DEBUG,  # 默认输出DEBUG及以上的数据
                 stream=True,
                 file=True):
        self.log_file_name = log_file_name
        self.min_level = min_level
        logging.Logger.__init__(self, self.log_file_name, level=self.min_level)
        if stream:
            self.__setStreamHandler__()
        if file:
            self.__setFileHandler__()

    def __setStreamHandler__(self, level=None):
        stream_handler = logging.StreamHandler()
        formatter = logging.Formatter('%(asctime)s %(filename)s[line:%(lineno)d] %(levelname)s %(message)s')
        stream_handler.setFormatter(formatter)
        if not level:
            stream_handler.setLevel(self.min_level)
        else:
            stream_handler.setLevel(level)
        self.addHandler(stream_handler)

    def __setFileHandler__(self, level=None):
        # 当前日志文件是由哪个日志对象打印出来的
        file_name = os.path.join(LOG_PATH, '{name}.log'.format(name=self.log_file_name))
        # 保存1年的日志
        file_handler = TimedRotatingFileHandler(filename=file_name, when='D', interval=1, backupCount=365, encoding='utf-8')
        file_handler.suffix = '%Y%m%d.log'
        if not level:
            file_handler.setLevel(self.min_level)
        else:
            file_handler.setLevel(level)
        formatter = logging.Formatter('%(asctime)s %(filename)s[line:%(lineno)d] %(levelname)s %(message)s')

        file_handler.setFormatter(formatter)
        self.file_handler = file_handler
        self.addHandler(file_handler)


if __name__ == '__main__':
    # 一些使用习惯:
    # 大多出情况下，只会打印info级别和error级别的日志。info代表正常情况，error代表异常情况。
    # 打印info级别日志时，创建日志对象指定名称为info，且日志对象名为log_info
    log_info = LogHandler('info')
    log_info.info('打印info日志')

    log_error = LogHandler('error')
    log_error.error('打印error日志')
```
# Python的序列化和反序列化



## 前言

在Python中，序列化一个对象是非常麻烦的事，`json.dumps`只能序列化dict，没法序列化对象。即便调用对象的``__dict__``函数，也会有问题，下面是一个简单案例：

```Python
import json


class Subject:
    def __init__(self, name: str = '', score: float = 0):
        self.name = name
        self.score = score


class Person:
    def __init__(self, username: str = '', age: int = 0, subject: Subject = None):
        self.username = username
        self.age = age
        self.subject = subject


subject = Subject(name='math', score=99)
person = Person('lhl', 8, subject=subject)
s = json.dumps(person)  # 序列化失败，因为对象中还嵌套对象
print(s)
```



## 实战

建议使用`jsonpickle`，使用前需要pip install。

```Python
import json

import jsonpickle


class Subject:
    def __init__(self, name: str = '', score: float = 0):
        self.name = name
        self.score = score


class Person:
    def __init__(self, username: str = '', age: int = 0, subject: Subject = None):
        self.username = username
        self.age = age
        self.subject = subject


subject = Subject(name='math', score=99)
person = Person('lhl', 8, subject=subject)
s = jsonpickle.encode(person)  # 序列化 {"py/object": "__main__.Person", "username": "lhl", "age": 8, "subject": {"py/object": "__main__.Subject", "name": "math", "score": 99}}
print(s)
obj = jsonpickle.decode(s)  # 反序列化
print(obj.username)
```

上述代码中，你可能会发现有`py/object`属性，这是为了反序列化时找到指定的类（搞不懂Python为什么需要这个属性，Java/Golang中都不需要一个属性指定对象）

如果我们想要得到对象序列化后纯正的Json字符串，在encode函数中传入参数unpicklable=False即可，代码如下：

```Python
import jsonpickle


class Subject:
    def __init__(self, name: str = '', score: float = 0):
        self.name = name
        self.score = score


class Person:
    def __init__(self, username: str = '', age: int = 0, subject: Subject = None):
        self.username = username
        self.age = age
        self.subject = subject


subject = Subject(name='math', score=99)
person = Person('lhl', 8, subject=subject)
s = jsonpickle.encode(person, unpicklable=False)
print(s)
```
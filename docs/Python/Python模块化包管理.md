# Python模块化包管理



## 前言

Python的模块化包管理类似Java的包管理。需要先知道一下概念：

1. 模块：一个后缀为.py的文件，叫做一个模块。
2. 包：一个包含很多.py文件的文件夹，通常该文件夹下有一个`__init__.py`文件。在pycharm中创建package的时候，相当于创建一个文件夹并在文件夹下面生成一个`__init__.py`文件。
3. 库：可能包含多个包。可以认为是一个完整项目的打包。



## 实战

使用pycharm创建一个空的python项目。分别思考下述情况的import语句该怎么写。核心是import的根路径是当前项目的根目录。通常建议一个文件夹下放一个`__init__.py`文件。

- 案例1

```Bash
a.py
b.py
# a.py
def func_a():
    print('func a')
# b.py
import a  # 为什么这么写
a.func_a()

# 另一种写法
from a import func_a  # 为什么这么写
func_a()
```

- 案例2

```Bash
a.py
p
--b.py
# b.py
def func_b():
    print('func b')
# a.py
import p.b  # 思考为什么这么写

p.b.func_b()    

# 也可以这么写
import p.b as pkg

pkg.func_b()
```

- 案例3

```Python
a.py
p
--p2
------b.py
# a.py
def func_a():
    print('func a')
# b.py
import a  # 思考为什么这么写

a.func_a()
```
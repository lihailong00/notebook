# Pycharm管理python项目

[toc]

## 写在前面

当我刚使用pycharm的时候，遇到过很多问题。virtualenv是什么？python的编译器（实际上是解释器）安装在哪里？python的依赖包放在哪里？pip是什么？

看了这篇[文章](https://blog.csdn.net/mukes/article/details/115905766)，我对pycharm管理python项目有了更深的理解。



## python代码如何运行

Python是编译+解释型的语言，执行的时候是由**Python解释器**逐行编译+解释，然后运行，因为在运行的过程中，需要编译+解释，所以Python的运行性能会低于编译型语言。

因此想要运行python代码，需要python解析器。我们去官网下载Python的安装包（安装包包含解释器、pip包等），并且指定安装路径。之后我们需要将**python解释器**的路径和**pip**的路径加入环境变量中，以便在任何路径都能访问它们。这里展示我的python解释器和pip的**目录**，仅供参考。

```
D:\mydoc\environment\python\
D:\mydoc\environment\python\Scripts
```



## 关于pip

pip其实就是python的一个包，只是它的特别之处在于可以安装和管理别的依赖包。

常用指令：

```shell
# 查看pip包的版本及位置
pip -V

# 修改pip的镜像源（通常是全局修改，对整个电脑的pip包都有效）
# pip会在%APPDATA%/pip 目录下查找pip.ini 文件，并写入数据
pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/
pip config set install.trusted-host mirrors.aliyun.com

# 安装包
pip install 包名
# 升级包
pip install -U 包名
# 获取更多帮助
pip --help
```



## 关于virtualenv

virtualenv是一个包，用来创建独立的Python虚拟环境，可以将每个项目与其他项目独立开来，互不影响，解决了依赖包版本冲突的问题。我通常基于virtualenv创建项目。

virtualenv创建的项目中会基于原始的python环境，重新创建新的python解释器和新的依赖包（依赖包也可以从全局继承，不过我通常不会这么做，因为可能导致依赖冲突），通常依赖包中有pip。

建议使用命令行的方式创建一个虚拟环境，这样会对virtualenv有更深的理解。

### Windows下运行虚拟环境

首先需要保证全局有python解释器并配置到环境变量中，同时全局有pip包。

1. 安装依赖包：`pip install virtualenv`
2. 创建虚拟环境：`virtualenv venv`，venv是虚拟文件的名字，之后会将依赖包等文件存放到venv中。
3. 激活虚拟环境：`.\venv\Scripts\activate`。如果想退出虚拟环境，输入：`deactivate`。deactivate和activate在同一目录下。
4. 


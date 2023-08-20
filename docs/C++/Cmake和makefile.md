# Cmake和makefile

[toc]

## 参考文档

[CMake是什么？有什么用？](https://blog.csdn.net/Torres_10/article/details/80371425)



## 前言

Cmake：是一个跨平台的编译（Build）工具。如果一个项目由几十个工程构成，他们之间的调用关系错综复杂，此时我们仅需要一份CMakeLists.txt文档，利用Cmake工具，完成工程的搭建。

makefile：就是上述提到的CMakeLists.txt文档。



## makefile 案例

使用时创建makefile文件，复制以下代码。

执行时在bash中输入`make`即可。

```bash
# 定义变量
CXX = g++
TARGET = main
# SRC 是当前目录所有.cpp文件
SRC = $(wildcard *.cpp)
# OBJ 是当前目录所有.cpp文件的前缀拼接.o形成的文件
OBJ = $(patsubst %.cpp, %.o, $(SRC))

CXXFLAGS = -c -Wall

# TARGET 依赖于 OBJ
$(TARGET): $(OBJ)
        # 如果 OBJ 比 TARGET 新
        $(CXX) -o $(TARGET) $(OBJ)

# $@ 表示规则中目标条件 %.o
# $^ 表示规则中所有依赖条件 %.cpp
# $< 表示规则中第一个依赖条件
%.o: %.cpp
        $(CXX) $(CXXFLAGS) $^ -o $@

# make clean 清空中间文件
.PHONY: clean
clear:
        rm -f *.o $(TARGET)
```


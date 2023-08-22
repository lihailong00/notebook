# 美团Spring项目结构

[toc]

## 分层思想

美团将项目主要分为这些模块：**api**、**api-impl**、**biz**、common、**dao**......其实和MVC的三层思想比较类似。



## 封装思想

函数的参数列表和返回值通常会被封装成一个对象，尤其是参数列表中参数数量大于5。



## 对象转换思想

api/controller层model的转换使用converter。

biz/service层model的转换使用assembler。



## API/Controller层


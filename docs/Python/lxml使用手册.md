# lxml使用手册

[toc]

## lxml库有什么用

lxml 库提供了一个 etree 模块，该模块专门用来解析 HTML/XML 文档。



## 一些坑

Chrome浏览器有时会自动增加tbody，导致无法解析到某个元素。**巨坑！**



## 基本使用方法

1. pycharm中安装lxml。

2. 参考如下案例：

   ```python
   from lxml import etree
   
   
   def main():
       # 定义html文本
       html_text = """
           <div>
               <div class="content">
                   操作系统
                   <br>
                   <font title="老师">
                       李老师
                   </font>
                   <font title="教室">
                       IV-A101
                   </font>
               </div>
           </div>
       """
   
       # 解析HTML文本
       tree = etree.HTML(html_text)
   
       # 案例1：获取 class="content" 标签中的 "操作系统" 文字
       # 步骤1：定位到 <div class="content"></div> 这【类】标签
       tags = tree.xpath('//div[@class="content"]')
       # 步骤2：遍历这类标签
       for tag in tags:
           # 这样获取到的文本可能有换行符和tab键，实际上不一定会出现这种情况
           # 【注意】xpath中的参数必须以 ./ 开头，而不是 / 或者 //
           name = tag.xpath('./text()')
           print(name)
   
       # 案例2：获取 class="content" 标签中的所有文字
       # 基于上述获得的tags
       for tag in tags:
           name = tag.xpath('.//text()')
           print(name)
   
   
   if __name__ == '__main__':
       main()
   
   ```




```python
from lxml import etree

def main():
    html_text = """
        <div>
            <div class="course">
                <div class="name">操作系统</div>
                <div class="teacher">李老师</div>
                <div class="room">IV-A401</div>
            </div>
            <div class="course">
                <div class="name">数据结构</div>
                <div class="teacher">张老师</div>
                <div class="room"></div>
            </div>
        </div>
    """

    # 要求：获取上述html文本的课程信息，如果课程的某个属性为空，也要将它显示出来
    tree = etree.HTML(html_text)
    courses = tree.xpath('//div[@class="course"]')

    # 这样写的缺点是没法显示课程 数据结构 的教室
    for course in courses:
        print(course.xpath('.//text()'))

    # 应该定位到所有的子div(属性为name 或 teacher 或 room)，如下
    for course in courses:
        # 获取到子div
        for data in course.xpath('./div'):
            print(data.xpath('./text()'))


if __name__ == '__main__':
    main()

```


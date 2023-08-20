# Java IO

[toc]

## 什么是流

可以它俩具象化为一根水管，负责传输数据，而流本身并不包含数据。



## 流的分类

1. 按照**流的方向**分类：

`inputstream`：开始端接着硬盘，结束端接着内存，“流”的方向是从硬盘到内存。

`outputstream`：开始端接着内存，结束端接着硬盘，“流”的方向是从内存到硬盘。



2. 按照**处理单位**分类：

   字节流：每次处理1个字节。

   字符流：每次处理1个字符，需要指定编码方式。

   

3. 按照**流的角色**分类：

   节点流：直接与数据源或数据目标进行交互，处理原始的字节或字符数据。

   处理流（也叫包装流）：基于已存在的流，为程序提供更强大的读写功能。



## 常用stream

> fileoutputstream
>
> fileinputstream：每次想内存中读入1B的数据。



## 小案例

案例1：文件的拷贝

```java
package org.example;

import java.io.*;

public class Main {
    public static void main(String[] args) {
        String startPath = "C:\\Users\\晓龙coding\\Desktop\\测试文件.txt";
        String endPath = "C:\\Users\\晓龙coding\\Desktop\\目标文件.txt";
        FileInputStream fileInputStream = null;
        FileOutputStream fileOutputStream = null;
        try {
            // 
            fileInputStream = new FileInputStream(startPath);
            fileOutputStream = new FileOutputStream(endPath);
            int readLen = 0;
            byte[] buf = new byte[1024];
            while ((readLen = fileInputStream.read(buf)) != -1) {
                fileOutputStream.write(buf, 0, readLen);
            }
        } catch (FileNotFoundException e) {
            throw new RuntimeException(e);
        } catch (IOException e) {
            throw new RuntimeException(e);
        } finally {
            try {
                fileInputStream.close();
                fileOutputStream.close();
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
        }
    }
}
```





`StringReader_`和`FileReader_` 是节点流
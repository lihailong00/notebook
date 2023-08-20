# Java时间类工具

[toc]

## 引入依赖

```xml
<dependency>
    <groupId>joda-time</groupId>
    <artifactId>joda-time</artifactId>
    <version>2.9.2</version>
</dependency>
```





## 案例演示

```java
// 获取当前时间，精确到毫秒
System.out.println(new DateTime());
// 自定义创建时间，需要精确到毫秒
System.out.println(new DateTime(2023, 1, 1, 12, 13, 12, 123));
// long类型的时间戳，显示距离1970-01-01T00:00:00Z的毫秒数
System.out.println(new DateTime(100));
// 解析字符串为时间（日期用-隔开、时间:隔开、毫秒用.隔开、日期与时间用T隔开、可以填写任意符合格式规范的前缀字符串）
System.out.println(new DateTime("2022-1-1T12:23:21.221"));
```


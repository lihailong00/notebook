# Python字符串处理

[toc]



## 子串是否存在于模板串

```python
str1 = 'hello world!'
sub = 'or'
print(sub in str1)  # True
```



## 子串出现的次数

```python
s = "星期一星期三"
print(s.count("星期"))  # 2
```





## 查找一个字符在给定字符串中的首次出现位置和最后一次出现的位置

```python
s = 'I have an apple, I have a pen. Oh ~ applepen'
print(s.find("apple"))  # 10
print(s.rfind("apple"))  # 36
print(s.find("has"))  # -1
```



## 字符串分割

```python
s = 'Welcome,this is my house,do you like it?'
print(s.split(','))  # ['Welcome', 'this is my house', 'do you like it?']
print(s.split(',', maxsplit=1))  # ['Welcome', 'this is my house,do you like it?']
print(s.rsplit(',', maxsplit=1))  # ['Welcome,this is my house', 'do you like it?']
print(s.split('bad'))  # ['Welcome,this is my house,do you like it?']
print(s.partition(','))  # 返回 tuple ('Welcome', ',', 'this is my house,do you like it?')
print(s.rpartition(','))  # ('Welcome,this is my house', ',', 'do you like it?')
```


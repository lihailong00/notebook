# go语言代码规范

[toc]

## 注释

1. 注释应该解释代码的作用**是什么**。
2. 注释应该解释代码**为什么**要有这个功能。
3. 注释应该解释代码是**怎么做**的。
4. 注释应该解释代码什么时候**会出错**。



## 命名规范

1. 缩略词全部大写。
   1. 正例：`ServerHTTP`
   2. 反例：`ServerHttp`
2. 函数名不能携带包名的上下文信息。
   1. 正例：http包中，函数名为`Server`。
   2. 反例：http包中，函数名为`ServerHTTP`。
3. 关于package命名
   1. 只有小写字母。
   2. 简短且包含一定的上下文信息。
   3. 不要与标准库同名。
   4. 使用单数。



## 控制流程

1. 保持最小缩进。

   1. 正例：

      ```go
      func Do1() error {
          err := Do2()
          if err != nil {
              return err
          }
          err := Do3()
          if err != nil {
              return err
          }
          return nil
      }
      ```

   2. 反例：

      ```go
      func Do1() error {
          err := Do2()
          if err == nil {
              err := Do3()
              if err == nil {
                  return nil
              }
              return err
          }
          return err
      }
      ```

      



## 异常处理

1. 简单错误

   1. 简单错误指的是仅出现一次的错误，且该错误不会出现在其他地方。
   2. 优先使用`errors.New`创建匿名变量表示简单错误。
   3. 使用`fmt.Errorf`格式化。

   ```go
   func check(req *Request, via []*Request) error {
       if len(via) >= 10 {
           return errors.New("Stop!")
       }
       return nil
   }
   ```

   
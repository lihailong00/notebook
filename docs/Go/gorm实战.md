# gorm 实战

```
mkdir gormlearn
cd gormlearn
go work init
mkdir gormtest
cd gormtest
go mod init test.com/gormtest
cd ../
go work use ./gormtest

```



在gormtest目录下：

```
// 安装mysql驱动
go get -u gorm.io/driver/mysql
// 安装gorm包
go get -u gorm.io/gorm
// 安装gin
go get -u github.com/gin-gonic/gin
```



gorm生态

![img](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/typora/926DB086CC9655D36C3EE85F3CBD2561.png)

# Gorm快速上手



## 原生GORM操作

使用SQLite演示操作。

gorm tag不用记，通常使用插件根据数据表生成对应的结构体。

```Go
package main

import (
    "fmt"
    "gorm.io/driver/sqlite"
    "gorm.io/gorm"
)

type User struct {
    Id       int64  `gorm:"primaryKey; autoIncrement:true; column:id; not null"`
    Username string `gorm:"column:username; not null; default:''"`
    Age      uint   `gorm:"column:age; not null; default:0"`
    Score           // 嵌套的类会被展平为一层
}

type Score struct {
    Chinese int `gorm:"column:chinese; not null; default:0"`
    Math    int `gorm:"column:math; not null; default:0"`
}

func main() {
    db, err := gorm.Open(sqlite.Open("test.db"), &gorm.Config{})
    if err != nil {
       panic("failed to connect database")
    }

    // 迁移 schema。一般我不习惯根据model生成数据表，而是根据数据表生成model。
    if err := db.AutoMigrate(&User{}); err != nil {
       fmt.Printf("auto migrate err: %s\n", err)
       return
    }

    // Create
    db.Create(&User{Username: "D42", Age: 100})

    // Read
    var product User
    db.First(&product, 1)                 // 根据整型主键查找
    db.First(&product, "code = ?", "D42") // 查找 code 字段值为 D42 的记录

    // Update - 将 product 的 price 更新为 200
    db.Model(&product).Update("Age", 200)
    // Update - 更新多个字段
    db.Model(&product).Updates(User{Age: 200, Username: "F42"})  // 仅更新非零值字段！！！
    db.Model(&product).Updates(map[string]interface{}{"Age": 200, "Username": "F42"})  // 可以更新零值字段

    // Delete - 删除 product
    db.Delete(&product, 1)
}
```



## 实战配置

https://gorm.io/zh_CN/gen/

通常使用gorm-gen生成model代码和操作数据库的代码。

根据gorm-gen的官方文档，我们可以编写如下代码。个人习惯将该代码放在``cmd/generate/generate.go``文件中。

```Go
package main

import (
    "code.lihailong.vip/longcoding/gorm-demo/demo2/biz/dal/model"
    "gorm.io/driver/mysql"
    "gorm.io/gen"
    "gorm.io/gorm"
)

func main() {
    g := gen.NewGenerator(gen.Config{
       OutPath:      "biz/dal/query",
       ModelPkgPath: "biz/dal/model",
       Mode:         gen.WithoutContext | gen.WithDefaultQuery | gen.WithQueryInterface, // generate mode

       FieldWithIndexTag: true,
       FieldWithTypeTag:  true,
       // 如果字段可以为null，则model类中的字段类型为指针。
       FieldNullable: true,

       // 个人建议数据库中的字段值不要设置为0值！
       // 有default值的字段，无法将字段值设置为零值(0或"")，这是gorm的设计理念之一。因为在go中，没有赋值的对象值通常是零值。
       // 但是有时候我们希望即便一个字段有默认值，也能将字段值设置为零值
       // 此时model类中需要生成指针类型的字段
       //FieldCoverable: true,
    })
    db, _ := gorm.Open(mysql.Open("root:123456@(127.0.0.1:3306)/web_project?charset=utf8mb4&parseTime=True&loc=Local"))
    g.UseDB(db)

    // 根据数据表生成go代码
    g.ApplyBasic(g.GenerateModel("user_info"))
    g.Execute()
}
```

开发习惯：

1. 数据库中的值不要设定为零值，例如：让1代表男生，2代表女生，0作为默认值，不要用。
2. 建议所有值都不要为null，且有default值。

## 数据表变动

1. 当我们给数据表增加一个字段时，我们只需要重新运行上述gen的代码即可。
2. 当我们修改数据表的一个字段名时，运行上述的gen代码后，引用原字段的代码会找不到对象，从而报错。这时候建议手动一个个修改。
3. 当我们删除数据表的一个字段时，运行gen代码后，也会让原字段的代码会找不到对象。还是建议手动一个个删除。
4. 不建议修改数据表名！

## CRUD

数据表结构如下：

```SQL
create table user
(
    id          bigint auto_increment
        primary key,
    username    varchar(30)   not null,
    password    varchar(100)  not null,
    age         int default 0 not null,
    hobby       varchar(100)  null,
    create_time datetime      not null,
    constraint idx_username
        unique (username),
    constraint idx_username_and_password
        unique (username, password)
);
```

首先要初始化数据库：

> biz/dal/initDB.go

```Go
package dal

import (
    "gorm.io/driver/mysql"
    "gorm.io/gorm"
)

var db *gorm.DB

func GetDB() *gorm.DB {
    return db
}

func InitDB() {
    var err error
    db, err = gorm.Open(mysql.Open("root:123456789@(127.0.0.1:3306)/web_project?charset=utf8mb4&parseTime=True&loc=Local"))
    if err != nil {
       panic(err.Error())
    }
}
```

> depend/depend.go

```Go
package depend

import (
    "code.lihailong.vip/longcoding/gorm_demo/biz/dal"
    "context"
)

func InitDepend(ctx context.Context) {
    // 初始化各种依赖
    dal.InitDB()
}
```

> main.go

```Go
package main

import (
    "code.lihailong.vip/longcoding/gorm_demo/biz/dal"
    "code.lihailong.vip/longcoding/gorm_demo/biz/dal/model"
    "code.lihailong.vip/longcoding/gorm_demo/biz/dal/query"
    "code.lihailong.vip/longcoding/gorm_demo/depend"
    "context"
    "fmt"
    "gorm.io/gorm/clause"
    "time"
)

func Str2Ptr(s string) *string {
    return &s
}

func main() {
    // 初始化代码
    ctx := context.Background()
    depend.InitDepend(ctx)

    // 业务代码
    //querySingleUser(ctx)
    queryUsers(ctx)
    //upsertUser(ctx)
    //createUser(ctx)
    //batchCreateUser(ctx)
    //upsertUser4(ctx)
    //updateUser1(ctx)
}

func querySingleUser(ctx context.Context) {
    // 按照条件，查找某个值是否存在
    username := "lhl1"
    u := query.Use(dal.GetDB()).User
    // 通过主键排序，拿到第一个
    user, err := u.WithContext(ctx).Where(u.Username.Eq(username)).First() // 除了First，还有Last(最后一个)、take(随机顺序)
    if err != nil {
       // 报错通常是找不到数据
       fmt.Printf("query user error: %s\n", err)
    } else {
       fmt.Printf("user=%+v\n", user)
    }
}

func queryUsers(ctx context.Context) {
    password := "1231"
    u := query.Use(dal.GetDB()).User
    users, err := u.WithContext(ctx).Where(u.Password.Eq(password)).Find()
    if err != nil {
       fmt.Printf("queryUsers error: %s\n", err)
    } else {
       if len(users) == 0 {
          fmt.Println("users is null")
          return
       }
       for idx, user := range users {
          fmt.Printf("idx=%d, user=%+v\n", idx, user)
       }
    }
}

// 创建一个全新的值
func createUser(ctx context.Context) {
    user := &model.User{
       Username:   "lhl",
       Password:   "123456",
       CreateTime: time.Now(),
    }

    u := query.Use(dal.GetDB()).User
    err := u.WithContext(ctx).Create(user)
    if err != nil {
       fmt.Printf("创建失败，原因:%s\n", err)
    } else {
       fmt.Printf("创建成功，user=%+v\n", user)
    }
}

func batchCreateUser(ctx context.Context) { // 要么全部创建成功，要么全部创建失败
    userList := []*model.User{
       {Username: "name1", Password: "111", CreateTime: time.Now()},
       {Username: "name2", Password: "222", CreateTime: time.Now()},
       //{ID: 1, Username: "name3", Password: "333", CreateTime: time.Now()},
    }
    u := query.Use(dal.GetDB()).User
    err := u.WithContext(ctx).Create(userList...)
    if err != nil {
       fmt.Printf("批量创建失败，原因:%s\n", err)
    } else {
       fmt.Println("批量创建成功")
    }
}

func upsertUser(ctx context.Context) {
    // 如果username存在，就更新password和hobby
    // 如果username不存在，就插入数据
    user := &model.User{
       Username:   "name1",
       Password:   "123",
       Hobby:      Str2Ptr("ctrl"),
       CreateTime: time.Now(),
    }
    u := query.Use(dal.GetDB()).User
    // 前提：一定要保证冲突的字段有【唯一索引】！！！
    err := u.WithContext(ctx).Clauses(clause.OnConflict{
       Columns: []clause.Column{{Name: string(u.Username.Desc().ColumnName())}}, // 1. 如果username发生冲突
       DoUpdates: clause.Assignments(map[string]interface{}{
          string(u.Password.Desc().ColumnName()): user.Password, // 注意，这里我没有直接写数据列名，防止数据表列名变更时这里出错
          string(u.Hobby.Desc().ColumnName()):    user.Hobby,
       }), // 2. 则按照map更新数据
    }).Create(user) // 3. 如果username没有发生冲突，则创建数据
    if err != nil {
       fmt.Printf("upsert error: %s", err)
       return
    }
    fmt.Println("upsert success!")
}

func upsertUser2(ctx context.Context) {
    // 如果username存在，则不做任何操作
    // 如果username不存在，则创建数据
    user := &model.User{
       Username:   "lhl",
       Password:   "123",
       Hobby:      Str2Ptr("ctrl"),
       CreateTime: time.Now(),
    }
    u := query.Use(dal.GetDB()).User
    // 前提：一定要保证冲突的字段有【唯一索引】！！！
    err := u.WithContext(ctx).Clauses(clause.OnConflict{
       Columns:   []clause.Column{{Name: string(u.Username.Desc().ColumnName())}}, // 1. 如果username发生冲突
       DoNothing: true,                                                            // 2. 则什么都不做
    }).Create(user) // 3. 如果username没有发生冲突，则创建数据

    if err != nil {
       fmt.Printf("upsert error: %s", err)
       return
    }
    fmt.Println("upsert success!")
}

func upsertUser3(ctx context.Context) {
    // 如果username存在，就更新除了主键以外的其他所有字段
    // 如果username不存在，就插入数据
    user := &model.User{
       Username:   "lhl",
       Password:   "123",
       Hobby:      Str2Ptr("唱跳rap篮球"),
       CreateTime: time.Now(),
    }
    u := query.Use(dal.GetDB()).User
    // 前提：一定要保证冲突的字段有【唯一索引】！！！
    err := u.WithContext(ctx).Clauses(clause.OnConflict{
       Columns:   []clause.Column{{Name: string(u.Username.Desc().ColumnName())}}, // 1. 如果username发生冲突
       UpdateAll: true,                                                            // 2. 则更新除了主键以外的所有字段
    }).Create(user) // 3. 如果username没有发生冲突，则创建数据
    if err != nil {
       fmt.Printf("upsert error: %s", err)
       return
    }
    fmt.Println("upsert success!")
}

func upsertUser4(ctx context.Context) {
    // 如果username和password都存在，就更新除了主键以外的其他所有字段
    // 否则就插入一条记录
    user := &model.User{
       Username:   "lhl",
       Password:   "123",
       Hobby:      Str2Ptr("rap"),
       CreateTime: time.Now(),
    }
    u := query.Use(dal.GetDB()).User
    // 前提：一定要保证冲突的字段有【唯一联合索引】或者2个【唯一索引】！！！
    err := u.WithContext(ctx).Clauses(clause.OnConflict{
       Columns:   []clause.Column{{Name: string(u.Username.Desc().ColumnName())}, {Name: string(u.Password.Desc().ColumnName())}}, // 1. 如果username+password发生冲突
       UpdateAll: true,                                                                                                            // 2. 则更新除了主键以外的所有字段
    }).Create(user) // 3. 如果username+password没有发生冲突，则创建数据
    if err != nil {
       fmt.Printf("upsert error: %s", err)
       return
    }
    fmt.Println("upsert success!")
}

func updateUser1(ctx context.Context) {
    // 如果username存在，则更新password
    // 否则提示更新失败
    user := &model.User{
       Username:   "lhl",
       Password:   "go",
       Hobby:      Str2Ptr("篮球"),
       CreateTime: time.Now(),
    }
    u := query.Use(dal.GetDB()).User
    resultInfo, err := u.WithContext(ctx).Where(u.Username.Eq(user.Username)).Updates(model.User{Password: user.Password})  // 零值不更新
    if err != nil {
       fmt.Printf("更新失败, err:%s\n", err)
       return
    } else {
       fmt.Printf("更新成功, resultInfo:%+v\n", resultInfo)
    }
}

func updateUser2(ctx context.Context) {
    // 如果username和password都存在，则更新hobby
    // 否则提示更新失败
    user := &model.User{
       Username:   "lhl",
       Password:   "go",
       Hobby:      Str2Ptr("篮球"),
       CreateTime: time.Now(),
    }
    u := query.Use(dal.GetDB()).User
    resultInfo, err := u.WithContext(ctx).Where(u.Username.Eq(user.Username)).Where(u.Password.Eq(user.Password)).Updates(model.User{Hobby: user.Hobby})  // 零值不更新
    if err != nil {
       fmt.Printf("更新失败, err:%s\n", err)
       return
    } else {
       fmt.Printf("成功更新%d条数据", resultInfo.RowsAffected)
    }
}
```
title: Gorm 使用
date: 2019-05-13 00:10:10 
author: Xavier
tags: 
    - Gorm
type: article
---

# Gorm 使用总结

什么是 ORM？为什么要⽤ ORM？

什么是 ORM ，即 Object-Relationl Mapping，它的作⽤是在关系型数据库和对象之间作⼀个映射，

这样，我们在具体的 操作数据库的时候，就不需要再去和复杂的 SQL 语句打交道，只要像平时操作对象⼀样操作它就可以了 。

ORM 解决的主要问题是对象关系的映射。域模型和关系模型分别是建⽴在概念模型的基础上的。

    域模型是⾯向对 象的
    关系模型是⾯向关系的

⼀般情况下，⼀个持久化类和⼀个表对应，类的每个实例对应表中的⼀条记录，

类的每个属性对应表的每个字段。
ORM 技术特点：

    提⾼了开发效率。
    
    由于 ORM 可以⾃动对 Entity 对象与数据库中的 Table 进⾏字段与属性的映射，所以我们实际 可能已经不需要⼀个专⽤的、庞⼤的数据访问层。
    
    ORM 提供了对数据库的映射，不⽤ sql 直接编码，能够像操作对象⼀样从数据库获取数据。

ORM 的缺点

ORM 的缺点是会牺牲程序的执⾏效率和会固定思维模式。

从系统结构上来看，采⽤ ORM 的系统⼀般都是多层系统，系统的层次多了，效率就会降低。ORM 是⼀种完全的 ⾯向对象的做法，⽽⾯向对象的做法也会对性能产⽣⼀定的影响。
2 ORM 操作数据

package main

import (
    "fmt"
    "github.com/jinzhu/gorm"
    _ "github.com/jinzhu/gorm/dialects/mysql"
)

// UserInfo 用户信息
//orm会默认根据结构体创建table ， orm采用的是linux命名方式  即小写加下划线，且会在名字后面加 s
//会创建user_infos 表
type UserInfo struct {
    gorm.Model
    ID     uint
    Name   string
    Gender string
    Hobby  string
}

//  truncate table 表名
//  数据库需要提前创建好  例如mygorm
//  parseTime是查询结果是否⾃动解析为时间
//  loc是MySQL的时区设置
//  charset是编码方式
func main() {
    fmt.Println("try open mysql connection....")
    db, err := gorm.Open("mysql", "root:123456@(localhost:3306)/mygorm?charset=utf8mb4&parseTime=True&loc=Local")
    if err != nil {
        panic(err)
    }
    fmt.Println("successful")
    defer db.Close()
    // 自动迁移
    //若该表不存在则创建该表，若该表存在且结构体发生变化则更新表结构
    db.AutoMigrate(&UserInfo{})
    u1 := UserInfo{gorm.Model{}, 1, "xiaozhu", "man", "playing"}
    // 创建记录
    result := db.Create(&u1)
    fmt.Println("result:", result.RowsAffected)
    // 查询
    var u = new(UserInfo)
    //查询一条记录
    db.First(u)
    fmt.Printf("First: %#v\n", u)
    //按照条件查询
    var uu UserInfo
    db.Find(&uu, "name=?", "xiaozhu")
    fmt.Printf("Find: %#v\n", uu)
    // 更新
    db.Model(&uu).Update("hobby", "sing")
    // 删除 ， 此处删除记录，是不会将数据表中的数据删除掉，而是deleted_at 会更新删除时间
    db.Delete(&uu)
}

    使用 gorm 必须要先创建好数据库
    gorm 会自动创建数据表，且表结构可以动态变化
    gorm 创建的表命名方式为 代码中结构体命名的转换， 例如 结构体命名为 UserInfo，则 table 会命名为 user_infos
    gorm 修改表结构非常的容易
    gorm 是完全面向对象的思想

3 模型定义

模型是标准的 struct，由 Go 的基本数据类型、实现了 Scanner 和 Valuer 接口的自定义类型及其指针或别名组成

例如：

type User struct {
  ID           uint
  Name         string
  Email        *string
  Age          uint8
  Birthday     *time.Time
  MemberNumber sql.NullString
  ActivatedAt  sql.NullTime
  CreatedAt    time.Time
  UpdatedAt    time.Time
}

自定义模型

遵循 GORM 已有的约定，可以减少您的配置和代码量。

type User struct {
   gorm.Model   // 内嵌
   Name         string
   Age          sql.NullInt64
   Birthday     *time.Time
   Email        string  `gorm:"type:varchar(100);uniqueIndex"`
   Role         string  `gorm:"size:255"`        // 设置字段大小为255
   MemberNumber *string `gorm:"unique;not null"` // 设置会员号（member number）唯一并且不为空
   Num          int     `gorm:"AUTO_INCREMENT"`  // 设置 num 为自增类型
   Address      string  `gorm:"index:addr"`      // 给address字段创建名为addr的索引
   IgnoreMe     int     `gorm:"-"`               // 忽略本字段
}

字段标签

声明 model 时，tag 是可选的，GORM 支持以下 tag： tag 名大小写不敏感，但建议使用 camelCase 风格
标签名 	说明
column 	指定 db 列名
type 	列数据类型，推荐使用兼容性好的通用类型，例如：所有数据库都支持 bool、int、uint、float、string、time、bytes 并且可以和其他标签一起使用，例如：not null、size, autoIncrement… 像 varbinary(8) 这样指定数据库数据类型也是支持的。在使用指定数据库数据类型时，它需要是完整的数据库数据类型，如：MEDIUMINT UNSIGNED not NULL AUTO_INSTREMENT
size 	指定列大小，例如：size:256
primaryKey 	指定列为主键
unique 	指定列为唯一
default 	指定列的默认值
precision 	指定列的精度
scale 	指定列大小
not null 	指定列为 NOT NULL
autoIncrement 	指定列为自动增长
autoIncrementIncrement 	自动步长，控制连续记录之间的间隔
embedded 	嵌套字段
embeddedPrefix 	嵌入字段的列名前缀
autoCreateTime 	创建时追踪当前时间，对于 int 字段，它会追踪秒级时间戳，您可以使用 nano/milli 来追踪纳秒、毫秒时间戳，例如：autoCreateTime:nano
autoUpdateTime 	创建 / 更新时追踪当前时间，对于 int 字段，它会追踪秒级时间戳，您可以使用 nano/milli 来追踪纳秒、毫秒时间戳，例如：autoUpdateTime:milli
index 	根据参数创建索引，多个字段使用相同的名称则创建复合索引，查看 索引 获取详情
uniqueIndex 	与 index 相同，但创建的是唯一索引
check 	创建检查约束，例如 check:age > 13，查看 约束 获取详情
<- 	设置字段写入的权限， <-:create 只创建、<-:update 只更新、<-:false 无写入权限、<- 创建和更新权限
-> 	设置字段读的权限，->:false 无读权限

- 	忽略该字段，- 无读写权限
   comment 	迁移时为字段添加注释
   关联标签
   标签 	描述
   foreignKey 	指定当前模型的列作为连接表的外键
   references 	指定引用表的列名，其将被映射为连接表外键
   polymorphic 	指定多态类型，比如模型名
   polymorphicValue 	指定多态值、默认表名
   many2many 	指定连接表表名
   joinForeignKey 	指定连接表的外键列名，其将被映射到当前表
   joinReferences 	指定连接表的外键列名，其将被映射到引用表
   constraint 	关系约束，例如：OnUpdate、OnDelete
   4 主键、表名、列名的约定
   主键（Primary Key）

GORM 默认会使⽤名为 ID 的字段作为表的主键。

 type User struct { 

 ID string // 名为`ID`的字段会默认作为表的主键 

 Name string 

 }

// 使⽤`AnimalID`作为主键 

type Animal struct { 

AnimalID int64 `gorm:"primary_key"` 

Name string 

Age int64 

} 

表名（Table Name）
表名默认就是结构体名称的复数，例如：

type User struct {} // 默认表名是 `users` 

// 将 User 的表名设置为 `profiles` 

func (User) TableName() string { 

    return "profiles" 

} 

func (u User) TableName() string { 

    if u.Role == "admin" { 
    
        return "admin_users" 
    
    } else { 
    
        return "users" 
    
    } 

} 

// 禁⽤默认表名的复数形式，如果置为 true，则 `User` 的默认表名是 `user` 

db.SingularTable(true) 

也可以通过 Table () 指定表名：

// 使⽤User结构体创建名为`deleted_users`的表 

db.Table("deleted_users").CreateTable(&User{}) 

var deleted_users []User 

db.Table("deleted_users").Find(&deleted_users) 

// SELECT * FROM deleted_users; 

db.Table("deleted_users").Where("name = ?", "jinzhu").Delete() 

// DELETE FROM deleted_users WHERE name = 'jinzhu'; 

GORM 还⽀持更改默认表名称规则：

gorm.DefaultTableNameHandler = func (db *gorm.DB, defaultTableName string) string { 
     return "prefix_" + defaultTableName;
} 

列名（Column Name）
列名由字段名称进⾏下划线分割来⽣成

type User struct { 

ID uint // column name is `id` 

Name string // column name is `name` 

Birthday time.Time // column name is `birthday` 

CreatedAt time.Time // column name is `created_at` 

} 

可以使⽤结构体 tag 指定列名：

type Animal struct { 

AnimalId int64 `gorm:"column:beast_id"` // set column name to `beast_id` 

Birthday time.Time `gorm:"column:day_of_the_beast"` // set co lumn name to `day_of_the_beast` 

Age int64 `gorm:"column:age_of_the_beast"` // set column name to `age_of_the_beast` 

} 

时间戳跟踪
CreatedAt

如果模型有 CreatedAt 字段，该字段的值将会是初次创建记录的时间。

db.Create(&user) // `CreatedAt`将会是当前时间 

// 可以使⽤`Update`⽅法来改变`CreateAt`的值 

db.Model(&user).Update("CreatedAt", time.Now()) 

UpdatedAt

如果模型有 UpdatedAt 字段，该字段的值将会是每次更新记录的时间。

db.Save(&user) // `UpdatedAt`将会是当前时间 

db.Model(&user).Update("name", "jinzhu") // `UpdatedAt`将会是当前时间 

DeletedAt

如果模型有 DeletedAt 字段，调⽤ Delete 删除该记录时，将会设置 DeletedAt 字段为当前时间，⽽

不是直接将记录从数据库中删除。
5 CURD
创建记录

user := User{Name: "Jinzhu", Age: 18, Birthday: time.Now()}

result := db.Create(&user) // 通过数据的指针来创建

user.ID             // 返回插入数据的主键
result.Error        // 返回 error
result.RowsAffected // 返回插入记录的条数

批量插入

要有效地插入大量记录，请将一个 slice 传递给 Create 方法。 将切片数据传递给 Create 方法，GORM 将生成一个单一的 SQL 语句来插入所有数据，并回填主键的值，钩子方法也会被调用。

var users = []User{{Name: "jinzhu1"}, {Name: "jinzhu2"}, {Name: "jinzhu3"}}
db.Create(&users)

for _, user := range users {
  user.ID // 1,2,3
}

使用 CreateInBatches 创建时，你还可以指定创建的数量，例如：

var users = []User{{name: "jinzhu_1"}, ...., {Name: "jinzhu_10000"}}

// 数量为 100
db.CreateInBatches(users, 100)

默认值

您可以通过标签 default 为字段定义默认值，如：

type User struct {
  ID   int64
  Name string `gorm:"default:galeone"`
  Age  int64  `gorm:"default:18"`
}

插入记录到数据库时，默认值 会被用于 填充值为 零值 的字段
查询
检索单个对象

GORM 提供了 First、Take、Last 方法，以便从数据库中检索单个对象。当查询数据库时它添加了 LIMIT 1 条件，且没有找到记录时，它会返回 ErrRecordNotFound 错误

// 获取第一条记录（主键升序）
db.First(&user)
// SELECT * FROM users ORDER BY id LIMIT 1;

// 获取一条记录，没有指定排序字段
db.Take(&user)
// SELECT * FROM users LIMIT 1;

// 获取最后一条记录（主键降序）
db.Last(&user)
// SELECT * FROM users ORDER BY id DESC LIMIT 1;

result := db.First(&user)
result.RowsAffected // 返回找到的记录数
result.Error        // returns error

// 检查 ErrRecordNotFound 错误
errors.Is(result.Error, gorm.ErrRecordNotFound)

    如果你想避免 ErrRecordNotFound 错误，你可以使用 Find，比如 db.Limit(1).Find(&user)，Find 方法可以接受 struct 和 slice 的数据。

First, Last 方法将按主键排序查找第一 / 最后一条记录，只有在用 struct 查询或提供 model value 时才有效，如果当前 model 没有定义主键，将按第一个字段排序，例如：

var user User
var users []User

// 可以
db.First(&user)
// SELECT * FROM `users` ORDER BY `users`.`id` LIMIT 1

// 可以
result := map[string]interface{}{}
db.Model(&User{}).First(&result)
// SELECT * FROM `users` ORDER BY `users`.`id` LIMIT 1

// 不行
result := map[string]interface{}{}
db.Table("users").First(&result)

// 但可以配合 Take 使用
result := map[string]interface{}{}
db.Table("users").Take(&result)

// 根据第一个字段排序
type Language struct {
  Code string
  Name string
}
db.First(&Language{})
// SELECT * FROM `languages` ORDER BY `languages`.`code` LIMIT 1

用主键检索

如果主键是数值类型，也可以通过 内联条件 传入主键来检索对象。如果主键是 string 类型，要小心避免 SQL 注入，查看 安全 获取详情

db.First(&user, 10)
// SELECT * FROM users WHERE id = 10;

db.First(&user, "10")
// SELECT * FROM users WHERE id = 10;

db.Find(&users, []int{1,2,3})
// SELECT * FROM users WHERE id IN (1,2,3);

如果主键是像 uuid 这样的字符串，您需要这要写：

db.First(&user, "id = ?", "1b74413f-f3b8-409f-ac47-e8c062e3472a")
// SELECT * FROM users WHERE id = "1b74413f-f3b8-409f-ac47-e8c062e3472a";

检索全部对象

// 获取全部记录
result := db.Find(&users)
// SELECT * FROM users;

result.RowsAffected // 返回找到的记录数，相当于 `len(users)`
result.Error        // returns error

条件
String 条件

// 获取第一条匹配的记录
db.Where("name = ?", "jinzhu").First(&user)
// SELECT * FROM users WHERE name = 'jinzhu' ORDER BY id LIMIT 1;

// 获取全部匹配的记录
db.Where("name <> ?", "jinzhu").Find(&users)
// SELECT * FROM users WHERE name <> 'jinzhu';

// IN
db.Where("name IN ?", []string{"jinzhu", "jinzhu 2"}).Find(&users)
// SELECT * FROM users WHERE name IN ('jinzhu','jinzhu 2');

// LIKE
db.Where("name LIKE ?", "%jin%").Find(&users)
// SELECT * FROM users WHERE name LIKE '%jin%';

// AND
db.Where("name = ? AND age >= ?", "jinzhu", "22").Find(&users)
// SELECT * FROM users WHERE name = 'jinzhu' AND age >= 22;

// Time
db.Where("updated_at > ?", lastWeek).Find(&users)
// SELECT * FROM users WHERE updated_at > '2000-01-01 00:00:00';

// BETWEEN
db.Where("created_at BETWEEN ? AND ?", lastWeek, today).Find(&users)
// SELECT * FROM users WHERE created_at BETWEEN '2000-01-01 00:00:00' AND '2000-01-08 00:00:00';

Struct & Map 条件

// Struct
db.Where(&User{Name: "jinzhu", Age: 20}).First(&user)
// SELECT * FROM users WHERE name = "jinzhu" AND age = 20 ORDER BY id LIMIT 1;

// Map
db.Where(map[string]interface{}{"name": "jinzhu", "age": 20}).Find(&users)
// SELECT * FROM users WHERE name = "jinzhu" AND age = 20;

// 主键切片条件
db.Where([]int64{20, 21, 22}).Find(&users)
// SELECT * FROM users WHERE id IN (20, 21, 22);

    注意 当使用结构作为条件查询时，GORM 只会查询非零值字段。这意味着如果您的字段值为 0、''、false 或其他 零值，该字段不会被用于构建查询条件，例如：

db.Where(&User{Name: "jinzhu", Age: 0}).Find(&users)
// SELECT * FROM users WHERE name = "jinzhu";

你可以使用 map 来构建查询条件，它会使用所有的值，例如：

db.Where(map[string]interface{}{"Name": "jinzhu", "Age": 0}).Find(&users)
// SELECT * FROM users WHERE name = "jinzhu" AND age = 0;

或查看 指定结构体查询字段 获取详情
指定结构体查询字段

当使用结构体进行查询时，你可以使用它的字段名或其 dbname 列名作为参数来指定查询的字段，例如：

db.Where(&User{Name: "jinzhu"}, "name", "Age").Find(&users)
// SELECT * FROM users WHERE name = "jinzhu" AND age = 0;

db.Where(&User{Name: "jinzhu"}, "Age").Find(&users)
// SELECT * FROM users WHERE age = 0;

内联条件

用法与 Where 类似

// SELECT * FROM users WHERE id = 23;
// 根据主键获取记录，如果是非整型主键
db.First(&user, "id = ?", "string_primary_key")
// SELECT * FROM users WHERE id = 'string_primary_key';

// Plain SQL
db.Find(&user, "name = ?", "jinzhu")
// SELECT * FROM users WHERE name = "jinzhu";

db.Find(&users, "name <> ? AND age > ?", "jinzhu", 20)
// SELECT * FROM users WHERE name <> "jinzhu" AND age > 20;

// Struct
db.Find(&users, User{Age: 20})
// SELECT * FROM users WHERE age = 20;

// Map
db.Find(&users, map[string]interface{}{"age": 20})
// SELECT * FROM users WHERE age = 20;

Not 条件

构建 NOT 条件，用法与 Where 类似

db.Not("name = ?", "jinzhu").First(&user)
// SELECT * FROM users WHERE NOT name = "jinzhu" ORDER BY id LIMIT 1;

// Not In
db.Not(map[string]interface{}{"name": []string{"jinzhu", "jinzhu 2"}}).Find(&users)
// SELECT * FROM users WHERE name NOT IN ("jinzhu", "jinzhu 2");

// Struct
db.Not(User{Name: "jinzhu", Age: 18}).First(&user)
// SELECT * FROM users WHERE name <> "jinzhu" AND age <> 18 ORDER BY id LIMIT 1;

// 不在主键切片中的记录
db.Not([]int64{1,2,3}).First(&user)
// SELECT * FROM users WHERE id NOT IN (1,2,3) ORDER BY id LIMIT 1;

Or 条件

db.Where("role = ?", "admin").Or("role = ?", "super_admin").Find(&users)
// SELECT * FROM users WHERE role = 'admin' OR role = 'super_admin';

// Struct
db.Where("name = 'jinzhu'").Or(User{Name: "jinzhu 2", Age: 18}).Find(&users)
// SELECT * FROM users WHERE name = 'jinzhu' OR (name = 'jinzhu 2' AND age = 18);

// Map
db.Where("name = 'jinzhu'").Or(map[string]interface{}{"name": "jinzhu 2", "age": 18}).Find(&users)
// SELECT * FROM users WHERE name = 'jinzhu' OR (name = 'jinzhu 2' AND age = 18);

您还可以查看高级查询中的 分组条件，它被用于编写复杂 SQL
选择特定字段

选择您想从数据库中检索的字段，默认情况下会选择全部字段

db.Select("name", "age").Find(&users)
// SELECT name, age FROM users;

db.Select([]string{"name", "age"}).Find(&users)
// SELECT name, age FROM users;

db.Table("users").Select("COALESCE(age,?)", 42).Rows()
// SELECT COALESCE(age,'42') FROM users;

还可以看一看 智能选择字段
Order

指定从数据库检索记录时的排序方式

db.Order("age desc, name").Find(&users)
// SELECT * FROM users ORDER BY age desc, name;

// 多个 order
db.Order("age desc").Order("name").Find(&users)
// SELECT * FROM users ORDER BY age desc, name;

db.Clauses(clause.OrderBy{
  Expression: clause.Expr{SQL: "FIELD(id,?)", Vars: []interface{}{[]int{1, 2, 3}}, WithoutParentheses: true},
}).Find(&User{})
// SELECT * FROM users ORDER BY FIELD(id,1,2,3)

Limit & Offset

Limit 指定获取记录的最大数量 Offset 指定在开始返回记录之前要跳过的记录数量

db.Limit(3).Find(&users)
// SELECT * FROM users LIMIT 3;

// 通过 -1 消除 Limit 条件
db.Limit(10).Find(&users1).Limit(-1).Find(&users2)
// SELECT * FROM users LIMIT 10; (users1)
// SELECT * FROM users; (users2)

db.Offset(3).Find(&users)
// SELECT * FROM users OFFSET 3;

db.Limit(10).Offset(5).Find(&users)
// SELECT * FROM users OFFSET 5 LIMIT 10;

// 通过 -1 消除 Offset 条件
db.Offset(10).Find(&users1).Offset(-1).Find(&users2)
// SELECT * FROM users OFFSET 10; (users1)
// SELECT * FROM users; (users2)

查看 Pagination 学习如何写一个分页器
Group & Having

type result struct {
  Date  time.Time
  Total int
}

db.Model(&User{}).Select("name, sum(age) as total").Where("name LIKE ?", "group%").Group("name").First(&result)
// SELECT name, sum(age) as total FROM `users` WHERE name LIKE "group%" GROUP BY `name`


db.Model(&User{}).Select("name, sum(age) as total").Group("name").Having("name = ?", "group").Find(&result)
// SELECT name, sum(age) as total FROM `users` GROUP BY `name` HAVING name = "group"

rows, err := db.Table("orders").Select("date(created_at) as date, sum(amount) as total").Group("date(created_at)").Rows()
for rows.Next() {
  ...
}

rows, err := db.Table("orders").Select("date(created_at) as date, sum(amount) as total").Group("date(created_at)").Having("sum(amount) > ?", 100).Rows()
for rows.Next() {
  ...
}

type Result struct {
  Date  time.Time
  Total int64
}
db.Table("orders").Select("date(created_at) as date, sum(amount) as total").Group("date(created_at)").Having("sum(amount) > ?", 100).Scan(&results)

Distinct

从模型中选择不相同的值

db.Distinct("name", "age").Order("name, age desc").Find(&results)

Distinct 也可以配合 Pluck, Count 使用
Joins

指定 Joins 条件

type result struct {
  Name  string
  Email string
}
db.Model(&User{}).Select("users.name, emails.email").Joins("left join emails on emails.user_id = users.id").Scan(&result{})
// SELECT users.name, emails.email FROM `users` left join emails on emails.user_id = users.id

rows, err := db.Table("users").Select("users.name, emails.email").Joins("left join emails on emails.user_id = users.id").Rows()
for rows.Next() {
  ...
}

db.Table("users").Select("users.name, emails.email").Joins("left join emails on emails.user_id = users.id").Scan(&results)

// 带参数的多表连接
db.Joins("JOIN emails ON emails.user_id = users.id AND emails.email = ?", "jinzhu@example.org").Joins("JOIN credit_cards ON credit_cards.user_id = users.id").Where("credit_cards.number = ?", "411111111111").Find(&user)

Joins 预加载

您可以使用 Joins 实现单条 SQL 预加载关联记录，例如：

db.Joins("Company").Find(&users)
// SELECT `users`.`id`,`users`.`name`,`users`.`age`,`Company`.`id` AS `Company__id`,`Company`.`name` AS `Company__name` FROM `users` LEFT JOIN `companies` AS `Company` ON `users`.`company_id` = `Company`.`id`;

Scan

Scan 结果至 struct，用法与 Find 类似

type Result struct {
  Name string
  Age  int
}

var result Result
db.Table("users").Select("name", "age").Where("name = ?", "Antonio").Scan(&result)

// 原生 SQL
db.Raw("SELECT name, age FROM users WHERE name = ?", "Antonio").Scan(&result)

处理错误

GORM 的错误处理与常见的 Go 代码不同，因为 GORM 提供的是链式 API。

如果遇到任何错误，GORM 会设置 *gorm.DB 的 Error 字段，您需要像这样检查它：

if err := db.Where("name = ?", "jinzhu").First(&user).Error; err != nil {
  // 处理错误...
}

或者

if result := db.Where("name = ?", "jinzhu").First(&user); result.Error != nil {
  // 处理错误...
}

ErrRecordNotFound

当 First、Last、Take 方法找不到记录时，GORM 会返回 ErrRecordNotFound 错误。如果发生了多个错误，你可以通过 errors.Is 判断错误是否为 ErrRecordNotFound，例如：

// 检查错误是否为 RecordNotFound
err := db.First(&user, 100).Error
errors.Is(err, gorm.ErrRecordNotFound)

技术是开放的，我们的心态，更应是开放的。拥抱变化，向阳而生，努力向前行。

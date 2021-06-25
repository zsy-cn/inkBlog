title: mysql
date: 2019-05-12 10:10:10
author: Xavier
tags: - mysql
type: article

---

# mysql

$ docker pull mysql
$ docker run -itd --name mysql-test -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql
$ docker exec -it mysql bash

#登录 mysql
mysql -u root -p

#添加远程登录用户

```
CREATE USER 'gogateway'@'%' IDENTIFIED WITH mysql*native_password BY '123456';
GRANT ALL PRIVILEGES ON _._ TO 'gogateway'@'%';
```

上面的命令也可使用多个权限同时赋予和回收，权限之间使用逗号分隔

```
mysql> grant select,update,delete,insert,create on go_gateway.* to 'gogateway'@'%';
mysql> grant all on *.* to 'gogateway'@'%';
```

赋予 MySQL 用户权限

一个新建的 MySQL 用户没有任何访问权限，这就意味着你不能在 MySQL 数据库中进行任何操作。你得赋予用户必要的权限。以下是一些可用的权限：

    ALL: 所有可用的权限
    CREATE: 创建库、表和索引
    LOCK_TABLES: 锁定表
    ALTER: 修改表
    DELETE: 删除表
    INSERT: 插入表或列
    SELECT: 检索表或列的数据
    CREATE_VIEW: 创建视图
    SHOW_DATABASES: 列出数据库
    DROP: 删除库、表和视图

运行以下命令赋予"myuser"用户特定权限。

```
    mysql> GRANT <privileges> ON <database>.<table> TO 'myuser'@'localhost';
```

以上命令中，<privileges> 代表着用逗号分隔的权限列表。如果你想要将权限赋予任意数据库（或表），那么使用星号（\*）来代替数据库（或表）的名字。

例如，为所有数据库/表赋予 CREATE 和 INSERT 权限：

```
    mysql> GRANT CREATE, INSERT ON *.* TO 'myuser'@'localhost';
```

验证给用户赋予的全权限：

```
    mysql> SHOW GRANTS FOR 'myuser'@'localhost';
```

如果想立即看到结果使用
flush privileges ;
命令更新

查看权限：

```
#select host,user,plugin,authentication_string from mysql.user
```

host 为 % 表示不限制 ip localhost 表示本机使用 plugin 非 mysql_native_password 则需要修改密码

navicat 链接错误；

修改密码：

```
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'root';
```

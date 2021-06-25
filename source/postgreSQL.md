title: PostgreSQL
date: 2019-05-12 10:10:10
author: Xavier
tags: - PostgreSQL
type: article

---

# PostgreSQL 数据库

## 拉取镜像

docker pull postgres:<tag>

## 新建并启动 PostgreSQL 容器

docker run --name <contanerName> -e POSTGRES_PASSWORD=<yourPassword> -p hostPort:containerPort -d postgres:<tag>

docker run --name iot-pq -e POSTGRES_PASSWORD=iot -p 5432:5432 -d postgres

    –name 指定容器名称。
    -e 指定环境变量，POSTGRES_PASSWORD 是 postgres 的数据库密码。
    -p 指定宿主机和 Docker 容器端口映射，冒号前为宿主机端口号，另一个是容器端口号。
    -d 指明容易后台运行。

## 进入容器

    psql 命令可以进入数据库客户端连接数据库服务端并进行操作。
    PostgreSQL 数据库常用操作可参考 PostgreSQL 数据库常用操作。
    -h 指明数据库 IP 地址。
    -U 指定登录数据库用户。

docker exec -it <containerID> psql -h <postgresqlIp> -U postgres

docker exec -it 15df572789f6 psql -h localhost -U postgres

## createdb 和 dropdb 创建、删除数据库

    切换到pg用户
    su - pg
    创建数据库mydb
    createdb mydb
    删除数据库mydb
    dropdb mydb

createdb 和 dropdb 可以指定主机，用户名，密码等参数，用于连接 PostgreSQL 服务器，以上示例没有指定，默认使用操作系统当前用户 pg，而 pg 用户也是启动数据库服务的用户，因此有权限创建和删除数据库。

## 创建用户及数据库

    postgres=# CREATE USER iot WITH PASSWORD 'iot';
    CREATE ROLE
    postgres=# CREATE DATABASE iot OWNER iot;
    CREATE DATABASE

## 远程登录，远程登录配置参照：linux 下 postgresql 远程登录

     psql -hx.x.x.x -p5432 -W    #登录命令
     enter password

## 导出表结构

```sql
select table_name, column_name,
    case data_type
        when 'character varying' then data_type || '(' || character_maximum_length || ')'
        when 'numeric' then data_type || '(' || numeric_precision || ',' || numeric_scale || ')'
        else data_type
    end as data_type,
    case is_nullable
        when 'NO' then 'NOT NULL'
        else ''
    end as is_nullable
from information_schema.columns
where table_schema='public'
order by table_name, ordinal_position;
```

Postgresql 设置时区
greatau 2020-11-10 09:12:26 876 收藏
分类专栏： postgreSQL 文章标签： postgresql
版权

1. 查看时区

```sql
show time zone;
```

2. 查看时间

```sql
select now();
```

3. 查看支持的时区列表

```sql
select \* from pg_timezone_names;
```

4.设置成东八区 北京时间 UTC+8

```sql
set time zone 'PRC';
```

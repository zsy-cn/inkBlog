title: PostgreSQL
date: 2019-05-12 10:10:10 
author: Xavier
tags: 
    - PostgreSQL
type: article
---

# PostgreSQL 数据库

## 拉取镜像

docker pull postgres:<tag>

## 新建并启动 PostgreSQL 容器

docker run --name <contanerName> -e POSTGRES_PASSWORD=<yourPassword> -p hostPort:containerPort -d postgres:<tag>

docker run --name sws-iot-pq -e POSTGRES_PASSWORD=swsiot -p 18286:5432 -d postgres

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

## createdb和dropdb创建、删除数据库

    切换到pg用户
    su - pg
    创建数据库mydb
    createdb mydb
    删除数据库mydb
    dropdb mydb

createdb和dropdb可以指定主机，用户名，密码等参数，用于连接PostgreSQL服务器，以上示例没有指定，默认使用操作系统当前用户pg，而pg用户也是启动数据库服务的用户，因此有权限创建和删除数据库。

## 创建用户及数据库

    postgres=# CREATE USER swsiot WITH PASSWORD 'swsiot';
    CREATE ROLE
    postgres=# CREATE DATABASE swsiot OWNER swsiot;
    CREATE DATABASE

## 远程登录，远程登录配置参照：linux下postgresql远程登录

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

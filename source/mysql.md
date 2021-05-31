title: mysql
date: 2019-05-12 10:10:10 
author: Xavier
tags: 
    - mysql
type: article
---

# mysql

$ docker pull mysql
$ docker run -itd --name mysql-test -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql
$ docker exec -it mysql bash

#登录mysql
mysql -u root -p

#添加远程登录用户
CREATE USER 'liaozesong'@'%' IDENTIFIED WITH mysql_native_password BY 'Lzslov123!';
GRANT ALL PRIVILEGES ON *.* TO 'liaozesong'@'%';

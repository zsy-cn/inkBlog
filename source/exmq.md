title: emqx
date: 2019-05-13 00:10:10 
author: Xavier
tags: 
    - emqx
type: article
---

# emqx 修改dashboard 密码
用户名 admin 与默认密码 public
emqx_ctl admins passwd admin 密码
admins add <Username> <Password> <Tags> 创建 admin 账号
admins passwd <Username> <Password>     重置 admin 密码
admins del <Username>                   删除 admin 账号

# 快速入门

## Start emqx
```sh
./bin/emqx start
```

## Check Status
```sh
./bin/emqx_ctl status
```

## Stop emqx
```sh
./bin/emqx stop
```

EMQ X 启动，可以使用浏览器访问 http://localhost:18083 来查看 Dashboard。
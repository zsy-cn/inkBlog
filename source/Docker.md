title: Docker
date: 2019-05-13 00:10:10
author: Xavier
tags: - Docker
type: article

---

# Docker 环境搭建

## 安装 Docker

brew cask install docker 或者 [官网](https://docs.docker.com/docker-for-mac/release-notes/)下载安装包

## 登陆 Docker Registry

```
docker login --username=xxx registry.xxxx.com
```

Docker 会将 token 存储在~/.docker/config.json 文件中，从而作为拉取私有镜像的凭证

## 工具

### Kitematic

## Docker run --rm

在 Docker 容器退出时，默认容器内部的文件系统仍然被保留，以方便调试并保留用户数据。
但是，对于 foreground 容器，由于其只是在开发调试过程中短期运行，其用户数据并无保留的必要，因而可以在容器启动时设置--rm 选项，这样在容器退出时就能够自动清理容器内部的文件系统。示例如下：
docker run --rm ba-208

等价于

docker run --rm=true ba-208

显然，--rm 选项不能与-d 同时使用（或者说同时使用没有意义），即只能自动清理 foreground 容器，不能自动清理 detached 容器。

注意，--rm 选项也会清理容器的匿名 data volumes。

所以，执行 docker run 命令带--rm 命令选项，等价于在容器退出后，执行 docker rm -v。
参考链接：

https://docs.docker.com/engine/reference/run/

## run

docker run [OPTIONS] IMAGE [COMMOND] [ARGS...]

# OPTIONS 说明

    --name="容器新名字": 为容器指定一个名称；
    -d: 后台运行容器，并返回容器ID，也即启动守护式容器；
    -i：以交互模式运行容器，通常与 -t 同时使用；
    -t：为容器重新分配一个伪输入终端，通常与 -i 同时使用；
    -P: 随机端口映射；
    -p: 指定端口映射，有以下四种格式
          ip:hostPort:containerPort
          ip::containerPort
          hostPort:containerPort
          containerPort

常用 OPTIONS 补足：
--name：容器名字
--network：指定网络
--rm：容器停止自动删除容器

-i：--interactive,交互式启动
-t：--tty，分配终端
-v：--volume,挂在数据卷
-d：--detach，后台运行

--- （-w 在 run 中，也可直接使用）
在已运行的容器中运行命令
docker exec [OPTIONS] CONTAINER COMMAND [ARG…]
常用选项：
-d：--detach ，后台运行命令
-e, --env list 设置 env
-i, --interactive 启用交互式
-t, --tty 启用终端
-u, --user string 指定用户 (格式: <name|uid>[:<group|gid>])
-w, --workdir string 指定工作目录

6.docker -e

指定环境变量

-e XXX_XXX="xxxxxxxxxxx"

5.docker -u

指定执行命令时，所使用的用户，不指定时，默认以 root 用户执行。

-d, --detach=false # 后台运行容器，并返回容器 ID；
-i, --interactive=false # 以交互模式运行容器，通常与 -t 同时使用；
-t, --tty=false # 为容器重新分配一个伪输入终端，通常与 -i 同时使用；
-u, --user="" # 指定容器的用户
-a, --attach=[] # 登录容器（必须是以 docker run -d 启动的容器）
-w, --workdir="" # 指定容器的工作目录
-c, --cpu-shares=0 # 设置容器 CPU 权重，在 CPU 共享场景使用
-e, --env=[] # 指定环境变量，容器中可以使用该环境变量
-m, --memory="" # 指定容器的内存上限
-P, --publish-all=false # 指定容器暴露的端口
-p, --publish=[] # 指定容器暴露的端口
-h, --hostname="" # 指定容器的主机名
-v, --volume=[] # 给容器挂载存储卷，挂载到容器的某个目录
--volumes-from=[] # 给容器挂载其他容器上的卷，挂载到容器的某个目录
--cap-add=[] # 添加权限，权限清单详见：http://linux.die.net/man/7/capabilities
--cap-drop=[] # 删除权限，权限清单详见：http://linux.die.net/man/7/capabilities
--cidfile="" # 运行容器后，在指定文件中写入容器 PID 值，一种典型的监控系统用法
--cpuset="" # 设置容器可以使用哪些 CPU，此参数可以用来容器独占 CPU
--device=[] # 添加主机设备给容器，相当于设备直通
--dns=[] # 指定容器的 dns 服务器
--dns-search=[] # 指定容器的 dns 搜索域名，写入到容器的/etc/resolv.conf 文件
--entrypoint="" # 覆盖 image 的入口点
--env-file=[] # 指定环境变量文件，文件格式为每行一个环境变量
--expose=[] # 指定容器暴露的端口，即修改镜像的暴露端口
--link=[] # 指定容器间的关联，使用其他容器的 IP、env 等信息
--lxc-conf=[] # 指定容器的配置文件，只有在指定--exec-driver=lxc 时使用
--name="" # 指定容器名字，后续可以通过名字进行容器管理，links 特性需要使用名字
--net="bridge" # 容器网络设置:
bridge # 使用 docker daemon 指定的网桥
host # 容器使用主机的网络
container:NAME_or_ID > # 使用其他容器的网路，共享 IP 和 PORT 等网络资源
none # 容器使用自己的网络（类似--net=bridge），但是不进行配置
--privileged=false # 指定容器是否为特权容器，特权容器拥有所有的 capabilities
--restart="no" # 指定容器停止后的重启策略:
no # 容器退出时不重启
on-failure # 容器故障退出（返回值非零）时重启
always # 容器退出时总是重启
--rm=false # 指定容器停止后自动删除容器(不支持以 docker run -d 启动的容器)
--sig-proxy=true # 设置由代理接受并处理信号，但是 SIGCHLD、SIGSTOP 和 SIGKILL 不能被代理

持久化 docker 的镜像或容器的方法

Docker 的镜像和容器可以有两种方式来导出

    docker save #ID or #Name
    docker export #ID or #Name

docker save 和 docker export 的区别

    对于Docker Save方法，会保存该镜像的所有历史记录
    对于Docker Export 方法，不会保留历史记录，即没有commit历史
    docker save保存的是镜像（image），docker export保存的是容器（container）；
    docker load用来载入镜像包，docker import用来载入容器包，但两者都会恢复为镜像；
    docker load不能对载入的镜像重命名，而docker import可以为镜像指定新名称。

save 命令

docker save [options] images [images...]

示例
docker save -o nginx.tar nginx:latest
docker save -o iot-frontendv32.tar sws/iot-frontend:v0.0.32
或
docker save > nginx.tar nginx:latest
其中-o 和>表示输出到文件，nginx.tar 为目标文件，nginx:latest 是源镜像名（name:tag）

load 命令

docker load [options]

示例
docker load -i nginx.tar
或
docker load < nginx.tar
其中-i 和<表示从文件输入。会成功导入镜像及相关元数据，包括 tag 信息

export 命令

docker export [options] container

示例
docker export -o nginx-test.tar nginx-test

#导出为 tar

docker export #ID or #Name > /home/export.tar

其中-o 表示输出到文件，nginx-test.tar 为目标文件，nginx-test 是源容器名（name）

import 命令

docker import [options] file|URL|- [REPOSITORY[:TAG]]

示例
docker import nginx-test.tar nginx:imp
或
cat nginx-test.tar | docker import - nginx:imp

转载请注明：[Xavier's Blog](https://zsy-cn.github.io) » [Docker](https://zsy-cn.github.io/Docker.html/)

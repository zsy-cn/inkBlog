title: Docker-Compose
date: 2019-05-16 00:10:10 
author: Xavier
tags: 
    - Docker
type: article
---

```
version: '3'
services:
 product:
 build: ./product
 volumes:
 - ./product:/usr/src/app
 ports:
 - 5001:80
​
 web:
 image: php:apache
 volumes:
 - ./web:/var/www/html
 ports:
 - 5000:80
 depends_on:
 - product
```

第一行的version: '3'并不是指定你安装的Docker Compose的版本， 实际上他和 Docker Compose的版本并无直接关系， 它指明的是你的compose file是使用版本3的格式来进行编写的。

​第二行开始就是我们docker-compose文件的主要内容， 它启动了两个服务： product和web。

​对于product服务，我们先使用bulid构建一个镜像， 服务除了可以基于指定的镜像，还可以基于一份 Dockerfile，它可以指定 Dockerfile 所在文件夹的路径， 在本例中为docker-compose.yml所在目录下的product文件夹。之后，我们使用volumes 挂载一个目录， 最后，我们使用ports进行端口映射， 本例是将容器的80端口映射到宿主机的5001端口之上。

​对于web服务， 它与product服务类似， 不同之处在于它直接使用了image指定了要使用的镜像， 注意最后的depends_on，一般项目容器启动的顺序是有要求的，如果直接从上到下启动容器，必然会因为容器依赖问题而启动失败，所以在yml文件中我们通过depend_on标签来先启动product服务， 解决容器的依赖。

转载请注明：[Xavier's Blog](https://zsy-cn.github.io) » [Docker-Compose](https://zsy-cn.github.io/Docker-Compose.html/)
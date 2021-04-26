title: Docker
date: 2019-05-13 00:10:10 
author: Xavier
tags: 
    - Docker
type: article
---

# Docker环境搭建
## 安装Docker
brew cask install docker 或者 [官网](https://docs.docker.com/docker-for-mac/release-notes/)下载安装包
## 登陆Docker Registry
```
docker login --username=xxx registry.xxxx.com
```
Docker会将token存储在~/.docker/config.json文件中，从而作为拉取私有镜像的凭证
## 工具
### Kitematic

转载请注明：[Xavier's Blog](https://zsy-cn.github.io) » [Docker](https://zsy-cn.github.io/Docker.html/)
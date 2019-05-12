title: Golang开发环境搭建
date: 2019-05-13 00:10:10 
author: Xavier
tags: 
    - Golang
type: article
---

# 安装GO
brew install go 或者 [官网](https://golang.org)下载安装包

# 配置Go环境变量GOPATH和GOBIN
1. 打开终端
```
cd ~
```

2. 查看是否有.bash_profile文件
```
ls -all
```

3. 有则跳过此步，没有则：
```
touch .bash_profile
```

4. 打开.bash_profile文件：
```
open -e .bash_profile
```

5. 自定义GOPATH和GOBIN位置：
```
export GOPATH=/Users/.../Go
export GOBIN=$GOPATH/bin
export PATH=$PATH:$GOBIN
```

6. 编译.bash_profile文件：
```
source .bash_profile
```

7. 查看Go环境变量：
```
go env
```

转载请注明：[Xavier's Blog](https://zsy-cn.github.io) » [Golang开发环境搭建](https://zsy-cn.github.io/Golang开发环境搭建.html/)
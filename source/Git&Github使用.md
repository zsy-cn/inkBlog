title: Git&Github 使用
date: 2019-05-13 00:10:10
author: Xavier
tags: - Git - Github - 版本控制
type: article

---

## Git 环境搭建

### 安装 Git

brew install git

### 设置 Git 信息

```
git config --global user.name "XXX"
git config --global user.email XXX
```

## Github 环境搭建

### 安装 GitHub Desktop

[官网](https://desktop.github.com/)下载安装

### 配置 SSH

```
ssh-keygen -t rsa -C "邮箱"
一直Enter，共三步
cd ~/.ssh
cat id_rsa.pub
```

复制出来，贴到 Github 的 SSH 处

### 修改提交方式为 SSH

```
$ git config --global url."git@mygitlab.com:".insteadOf "http://mygitlab.com/"

// 其实就是在 `.gitconfig 增加了配置`
$ cat ~/.gitconfig

[url "git@mygitlab.com:"]
    insteadOf = http://mygitlab.com/
```

由于 git 的仓库 进行初始化 的时候配置的是 https 的方式，

在 git push 的时候每次都要输入用户名 跟 密码。非常的不方便，

究其原因，该配置是在 ./git/config 文件中配置的

https 方式：

```
[core]
	repositoryformatversion = 0
	filemode = false
	bare = false
	logallrefupdates = true
	symlinks = false
	ignorecase = true
	hideDotFiles = dotGitOnly
[remote "origin"]
	url = https://git.coding.net/coder/DMP.git
	fetch = +refs/heads/*:refs/remotes/origin/*
[branch "dev"]
	remote = origin
	merge = refs/heads/dev
```

git 方式：

```
[core]
	repositoryformatversion = 0
	filemode = false
	bare = false
	logallrefupdates = true
	symlinks = false
	ignorecase = true
	hideDotFiles = dotGitOnly
[remote "origin"]
	url = git@git.coding.net:coder/DMP.git
	fetch = +refs/heads/*:refs/remotes/origin/*
[branch "dev"]
	remote = origin
	merge = refs/heads/dev
[gui]
	wmstate = normal
	geometry = 835x475+182+182 185 214
```

只要将

```
https: 
[remote "origin"]
   url = https://git.coding.net/asun_coder/DMP_JavaBackground.git
  fetch = +refs/heads/*:refs/remotes/origin/*
```

转换成

```
ssh:
[remote "origin"]
  url = git@git.coding.net:asun_coder/DMP_JavaBackground.git
  fetch = +refs/heads/*:refs/remotes/origin/*
```

```
git push --mirror github
```

转载请注明：[Xavier's Blog](https://zsy-cn.github.io) » [Git&Github 使用](https://zsy-cn.github.io/Git&Github使用.html/)

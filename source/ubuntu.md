title: Ubuntu
date: 2019-05-12 10:10:10
author: Xavier
tags: - Ubuntu
type: article

---

# 系统清理篇

系统更新

安装完系统之后，需要更新一些补丁。Ctrl+Alt+T 调出终端，执行一下代码:

sudo apt-get update
sudo apt-get upgrade

卸载 libreOffice

libreoffice 事 ubuntu 自带的开源 office 软件，体验效果不如 windows 上的 office，于是选择用 WPS 来替代

sudo apt-get remove libreoffice-common

删除 Amazon 的链接

sudo apt-get remove unity-webapps-common

删除不常用的软件

sudo apt-get remove thunderbird totem rhythmbox empathy brasero simple-scan gnome-mahjongg aisleriot

sudo apt-get remove gnome-mines cheese transmission-common gnome-orca webbrowser-app gnome-sudoku landscape-client-ui-install

sudo apt-get remove onboard deja-dup

做完上面这些，系统应该干净了，下面我们来安装一些必要的软件。
Ubuntu 在卸载软件时，其依赖包不会卸载，可以通过命令来删除，

清理旧版本软件缓存，输入：sudo apt-get autoclean

清理所有软件缓存，输入：sudo apt-get clean

删除系统不再使用的孤立软件，输入：sudo apt-get autoremove

# 更换 Ubuntu 的软件源

对于 Ubuntu 系统， 不同的版本的源都不一样，每一个版本都有自己专属的源。 而对于 Ubuntu 的同一个发行版本，它的源又分布在全球范围内的服务器上。Ubuntu 默认使用的官方源的服务器在欧洲，从国内访问速度很慢。国内的阿里、网易以及一些重点高校也都有 Ubuntu 的源，所以在装完 Ubuntu 系统后最好把官方源更换为国内的源。
这里我将告诉大家如何更换为国内的源：

step 1: 首先看看国内有哪些源，点我查看
我们选择阿里云源与清华大学源（其他源都行），将它们的 Ubuntu 源的服务器地址先复制下来，下面会用到。
阿里云源: http://mirrors.aliyun.com/ubuntu/
清华源: http://mirrors.tuna.tsinghua.edu.cn/ubuntu/

step 2: 获取 Ubuntu 代号
Ubuntu 每个发行版本都有自己的代号，我们要通过我们电脑上 Ubuntu 的代号去找对应的源，Ctrl+Alt+T 打开终端，执行以下命令：

```
lsb_release -a
```

然后会得到我们自己的 Ubuntu 的版本信息 ，最后一栏 codename 后面的就是我们自己的 Ubuntu 的代号。比如我安装的是 Ubuntu 18.04.1，查出来的代号就是 bionic.

step 3: 修改源文件 sources.list

## 清华大学源

https://mirror.tuna.tsinghua.edu.cn/help/ubuntu/

```
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted universe multiverse
```

deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ trusty main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ trusty-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ trusty-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ trusty-security main restricted universe multiverse

Ubuntu 的源存放在在 /etc/apt/ 目录下的 sources.list 文件中，修改前我们先做个备份，在终端中执行以下命令：

```
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bcakup
```

然后执行下面的命令打开 sources.list 文件，清空里面的内容，把上面我们编辑好的国内的源复制进去，保存后退出。

```
sudo vim /etc/apt/sources.list
```

step 4: 更新软件列表和升级
在终端上执行以下命令更新软件列表，检测出可以更新的软件：

```
sudo apt-get update
```

在终端上执行以下命令进行软件更新：

```
sudo apt-get upgrade
```

# 开启 ssh

## 安装 ssh

```
sudo apt-get install ssh
```

再使用查看 ssh 安装情况

```
dpkg -l | grep ssh
```

启动

```
sudo /etc/init.d/ssh start
```

然后查看状态

```
/etc/init.d/ssh status or ps -e | grep sshd
```

## 开机自启动

1、Ctrl + Alt + T 打开终端；

2、终端输入命令

```
gnome-session-properties
```

然后弹出窗口“ 启动应用程序首选项” ；

3、点击右侧 “添加” 按钮；

4、输入描述，选择软件路径（选择运行的软件）。

重启系统后，就会打开你添加自启动的软件。

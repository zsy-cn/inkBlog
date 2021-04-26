title: Mac 安装 putty
date: 2019-05-13 00:10:10 
author: Xavier
tags: 
    - Mac
    - putty
type: article
---

# 安装 Xcode
# 安装 macports
https://distfiles.macports.org/MacPorts/

在macports官网下载对应版本的macports安装文件，比如我是OS X EI Capitan就下载MacPorts-2.3.4-10.11-ElCapitan，格式为“包名-版本号-苹果系统版本号-具体系统名称”

macOs源于FreeBSD，ports是FreeBSD的一种包管理方式，其功用类似brew。

安装macports的过程很慢，请耐心等待。
FreeBSD 的 Ports 系统
什么是 Ports 系统

简单的讲，一个 port 就是一个被移植到了 FreeBSD 上的软件。所有这些软件的集合，加上 FreeBSD 处理这些软件的各种工具，就是 Ports 系统。
Ports 系统有什么用

每一个被移植到 FreeBSD 上的软件（就是 Port），都能通过 Ports 系统中的工具方便有序的安装，升级，卸载。而且符合 FreeBSD 系统对应用软件施加的各种规范。免去了你到处寻找软件，自己编译，安装，升级的麻烦。借助这些 ports 维护者的努力，你也不用担心这些软件与系统不兼容导致无法安装升级等等。

# 找不到 `port` 命令处理
port 安装到了 /opt/local/bin/port
这个路经默认不加入zsh
手动添加以下路径
```sh
open ~/.zshrc
```
```sh
# User configuration
export PATH=$PATH:/opt/local/bin
```

# MacPorts使用

更新ports tree和MacPorts版本，强烈推荐第一次运行的时候使用-v参数，显示详细的更新过程。
sudo port -v selfupdate

搜索索引中的软件
port search name

安装新软件
sudo port install name

卸载软件
sudo port uninstall name

查看有更新的软件以及版本
port outdated

升级可以更新的软件
sudo port upgrade outdated

# 更新 ports
```
sudo port -v selfupdate
```

# 安装 putty
```
sudo port install putty
```
安装putty后执行putty报没有这个命令，全盘查找也找不到可执行文件putty，只在putty本应存在的目录找到puttygen、 plink、psftp，看来是没有生成putty。（用brew安装也会有同样的问题）

上putty官网下载源码编译安装

sudo ./configure

第一步./configure报错如下

'configure' was unable to find either the GTK 1 or GTK 2 libraries on

your system. Therefore, PuTTY itself and the other GUI utilities will

not be built by the generated Makefile: only the command-line tools

such as puttygen, plink and psftp will be built.

报错信息跟之前看到的状况吻合，only the command-line tools such as puttygen, plink and psftp will be built.

只有puttygen、 plink、psftp这些命令行工具会生成，GUI utilities不会生成。

着手解决缺少GTK库的问题

sudo port install gtk1

sudo port install gtk2

执行完成后再次sudo port install putty，这次OK了，有可执行文件putty了，但是执行putty没什么反应，不弹图形界面。

# 安装 XQuartz
⑥Download and Install X11 (XQuartz)

http://xquartz.macosforge.org/landing/

上一步不弹图形界面是因为没有底层绘图支持，最后一步，安装底层绘图支持--X11 (XQuartz)。

安装以后，再在终端执行putty就可以弹出图形界面了。

Quartz是位于Mac OS X的Darwin核心之上的绘图层，有时候也认为是CoreGraphics。Quartz直接地支援Aqua，借由显示2D绘图图形来建立使用者接口，包含即时绘制（rendering）和次像素（sub-pixel）精准的反锯齿。
共有两种元件来组成Quartz：
Quartz Compositor合成视窗系统，管理和合成幕后视窗影像来建立Mac OS X使用者接口Quartz 2D以PDF的规范为基础的图形函式库，用来绘制二维文字和图形Quartz可以使用AltiVec来加速，以及透过AGP显卡上的GPU支援的硬件绘图。这像技术在Mac OS X Tiger上被扩充为Core Image和Core Video提供即时的视讯和图片的操作。

# (putty:46660): Gtk-WARNING **: 10:39:00.137: cannot open display: :0
需要打开 XQuartz 软件才能 运行 putty 命令图形界面
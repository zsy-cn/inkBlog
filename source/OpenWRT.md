title: OpenWRT讲解
date: 2019-05-12 10:10:10 
author: Xavier
tags: 
    - OpenWRT
    - Linux
type: article
---

# [教程](#教程)

https://www.right.com.cn/forum/thread-179557-1-1.html

# 安装依赖
sudo apt-get install gcc g++ binutils patch bzip2 flex bison make autoconf gettext texinfo unzip sharutils libncurses5-dev ncurses-term zlib1g-dev gawk asciidoc libz-dev git-core uuid-dev libacl1-dev liblzo2-dev pkg-config libc6-dev curl libxml-parser-perl ocaml-nox libssl-dev libstdc++6 lib32stdc++6（ubuntu16.04）

make package/xxxxx/compile V=99 

xxxxx就是你需要单独编译的程序。编译完成后去bin/ramips/packages里面找到对应的ipk，上传到板子，opkg install就可以了。

# opkg使用
```
update 下载服务器上可用的软件包列表
upgrade <包名> 升级软件包
install <包名> 安装软件包
configure <包名> 配置某一个软件包
remove <包名> 卸载软件包
info [pkg|regexp] 显示出指定软件包的信息
```

# OpenWrt配置opkg.conf
```
root@OpenWrt:/etc# cat opkg.conf 
dest root /
dest ram /tmp
lists_dir ext /var/opkg-lists
option overlay_root /overlay
src/gz barrier_breaker_base http://192.168.202.110/base
```
```
root@OpenWrt:/etc# opkg update
root@OpenWrt:/etc# opkg install nano
root@OpenWrt:/etc# opkg remove nano
```

# 搭建自己的openwrt opkg feed源服务器
将ipk包放进http的目录（如mypakcage）后，还需要Packages和Packages.gz两个文件，openwrt sdk下使用scripts/ipkg-make-index.sh这个脚本生成Packages文件

```
#先设置PATH路径，否则创建过程会提示mkhash出错
export PATH=”/work/oepnwrt/staging_dir/host/bin:$PATH”

#使用scripts/ipkg-make-index.sh这个脚本生成Packages文件
./scripts/ipkg-make-index.sh /tmp/yourpakdir > /tmp/Packages
gzip -9c /tmp/Packages > /tmp/Packages.gz
```

将Packages、Packages.gz拷贝到http的mypackage目录
在openwrt中的/etc/opkg/distfeeds.conf中添加你的http服务器地址与目录后执行opkg update之后，就可以通过opkg install下载你自己的包源


转载请注明：[Xavier's Blog](https://zsy-cn.github.io) » [OpenWRT讲解](https://zsy-cn.github.io/OpenWRT-awesome.html/)
title: OpenWRT 讲解
date: 2019-05-12 10:10:10
author: Xavier
tags: - OpenWRT - Linux
type: article

---

# [教程](#教程)

https://www.right.com.cn/forum/thread-179557-1-1.html

# 安装依赖

sudo apt-get install gcc g++ binutils patch bzip2 flex bison make autoconf gettext texinfo unzip sharutils libncurses5-dev ncurses-term zlib1g-dev gawk asciidoc libz-dev git-core uuid-dev libacl1-dev liblzo2-dev pkg-config libc6-dev curl libxml-parser-perl ocaml-nox libssl-dev libstdc++6 lib32stdc++6（ubuntu16.04）

make package/xxxxx/compile V=99

xxxxx 就是你需要单独编译的程序。编译完成后去 bin/ramips/packages 里面找到对应的 ipk，上传到板子，opkg install 就可以了。

# opkg 使用

```
update 下载服务器上可用的软件包列表
upgrade <包名> 升级软件包
install <包名> 安装软件包
configure <包名> 配置某一个软件包
remove <包名> 卸载软件包
info [pkg|regexp] 显示出指定软件包的信息
```

# OpenWrt 配置 opkg.conf

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

# 搭建自己的 openwrt opkg feed 源服务器

将 ipk 包放进 http 的目录（如 mypakcage）后，还需要 Packages 和 Packages.gz 两个文件，openwrt sdk 下使用 scripts/ipkg-make-index.sh 这个脚本生成 Packages 文件

```
#先设置PATH路径，否则创建过程会提示mkhash出错
export PATH=”/work/oepnwrt/staging_dir/host/bin:$PATH”

#使用scripts/ipkg-make-index.sh这个脚本生成Packages文件
./scripts/ipkg-make-index.sh /tmp/yourpakdir > /tmp/Packages
gzip -9c /tmp/Packages > /tmp/Packages.gz
```

将 Packages、Packages.gz 拷贝到 http 的 mypackage 目录
在 openwrt 中的/etc/opkg/distfeeds.conf 中添加你的 http 服务器地址与目录后执行 opkg update 之后，就可以通过 opkg install 下载你自己的包源

## 开机自启动

root@xxx:~# cat /etc/init.d/speech_once
#!/bin/sh /etc/rc.common

START=98

USE_PROCD=1
PROG=/usr/bin/speech_once

start_service() {
　　 procd_open_instance
　　 procd_set_param command "$PROG"
　　 procd_set_param respawn 0
　　 procd_close_instance
}

stop_service(){
　　 service_stop "$PROG"
}

其中，procd_set_param respawn 0，respawn 设置进程挂掉自动重启

    通常的嵌入式系统均有一个守护进程，该守护进程监控系统进程的状态，如果某些系统进程异常退出，将再次启动这些进程。procd 就是这样一个进程，它是使用C语言编写的，一个新的 OpenWrt 进程管理服务。它通过init脚本来将进程信息加入到 procd 的数据库中来管理进程启动，这是通过ubus总线调用来实现，可以防止进程的重复启动调用。

procd 的进程管理功能主要包含 3 个部分。

    reload_config，检查配置文件是否发生变化，如果有变化则通知 procd 进程。
    procd，守护进程，接收使用者的请求，增加或删除所管理的进程，并监控进程的状态，如果发现进程退出，则再次启动进程。
    procd.sh，提供函数封装procd提供系统总线方法，调用者可以非常便利的使用procd 提供的方法。

root@zihome:/etc/init.d# cat /var/run/config.md5
b9f0f9179b5b407ff85430da41877ba3 /var/run/config.check/dhcp
ce8e9fffa77a896c43cdeb2b83dede88 /var/run/config.check/dropbear
92a19521dd5c8cacd08e64e2785d94b1 /var/run/config.check/firewall
736bccfcc298fcdac1016da5f58518e1 /var/run/config.check/fstab
751c2e2ff80f2952b4c866fbc1d52f7d /var/run/config.check/guide
b9111c52e6db67b7bf550d5701a44e5f /var/run/config.check/htpdate
bad4e7384734245500ed9c081e0009f5 /var/run/config.check/igmpproxy
645f0a3cbfd1cf565b920dfd897af0c5 /var/run/config.check/network
3165f4af5f45e3209f9e20e28a78a76a /var/run/config.check/rpcd
87701d34b6de4ec46e6cb5edf1ac5f74 /var/run/config.check/system
6ec4ee52ca677ac87ff2469607a8cf9f /var/run/config.check/uhttpd
0ba26b28318a2a2d879fdad87f8e8c56 /var/run/config.check/upnpd
775e2ac3058da3ea2e973b3747233b92 /var/run/config.check/webpage
009ba5a3f9130b74662822d36d3ecd37 /var/run/config.check/wifidev
741a39d373e3c6f2dbeff47ca145606b /var/run/config.check/wireless
af357fab720926629ad9ab2ff4f465e1 /var/run/config.check/zihome
e2e3d1e8532386200a708f0c6837830c /var/run/config.check/zqos

如果在自己的启动脚本中定义了 USE_PROCD 那就调用这些函数。在 rc.common 中重
新定义了 start 函数，相当于重载了这些函数。
函 数 含 义
start_service 向 procd 注册并启动服务，是将在 services 所管理对象里面增加了一项
stop_service 让 procd 解除注册，并关闭服务, 是将在 services 中的管理对象删除
service_triggers 配置文件或网络接口改变之后触发服务重新读取配置
service_running 查询服务的状态
reload_service 重启服务，如果定义了该函数，在 reload 时将调用该函数，否则再次调用 start 函数
service_started 用于判断进程是否启动成功
举例：

通常有两行内容是固定的，第一
行表示使用“/etc/rc.common”来解释脚本。
第二行内容设置 USE_PROCD 变量为 1，表示使
用 procd 来管理进程

（1）pprocd_open_instance 开始增加一个服务实例。

（2）procd_set_param 设置服务实例的参数值，通常会有以下几种类型的参数。

    command: 服务的启动命令行。
    respawn: 进程意外退出的重启机制及策略，它需要有 3 个设置值。第一个设置为判断异常失败边界值（threshold），默认为3600秒，如果小于这个时间退出，则会累加重新启动次数，如果大于这个临界值，则将重启次数置 0。第二个设置为重启延迟时间（timeout），将在多少秒后启动进程，默认为5秒。第三个设置是总的失败重启次数（retry），是进程永久退出之前的重新启动次数，超过这个次数进程退出之后将不会再启动。默认为 5 次。也可以不带任何设置，那这些设置都是默认值。
    env：进程的环境变量。
    file：配置文件名，比较其文件内容是否改变。
    netdev：绑定的网络设备（探测 ifindex 更改）。
    limits：进程资源限制。
    每次只能使用一种类型参数，其后是这个类型参数的值。

（3）procd_close_instance 完成进程实例的增加。

vim /etc/init.d/zdetect

#!/bin/sh /etc/rc.common
USE_PROCD=1

START=88
STOP=92

start_service() {
procd_open_instance
procd_set_param command /usr/bin/zdetect
procd_set_param respawn
[ -e /proc/sys/kernel/core_pattern ] && {
procd_set_param limits core="unlimited"
}

        procd_close_instance

}

vim /etc/init.d/zboard

之后还需要在 rc.d 目录下做一个链接，启动时系统会按顺序启动 rc.d 目录下的脚本链接，对应执行 init.d 目录下的启动脚本。

        链接命令如下：ln -s ../init.d/wakeup /etc/rc.d/S98wakeup

        重启搞定~

转载请注明：[Xavier's Blog](https://zsy-cn.github.io) » [OpenWRT 讲解](https://zsy-cn.github.io/OpenWRT-awesome.html/)

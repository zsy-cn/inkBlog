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

```
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
```

vim /etc/init.d/zboard

之后还需要在 rc.d 目录下做一个链接，启动时系统会按顺序启动 rc.d 目录下的脚本链接，对应执行 init.d 目录下的启动脚本。

        链接命令如下：ln -s ../init.d/wakeup /etc/rc.d/S98wakeup

        重启搞定~

## openwrt 升级系统

md5 校验，确保下载的固件完整:

```sh
    root@OpenWrt:/tmp# wget http://downloads.openwrt.org/snapshots/trunk/ar71xx/md5sums
    root@OpenWrt:/tmp# md5sum -c md5sums 2> /dev/null | grep OK
    openwrt-ar71xx-generic-tl-wr2543-v1-squashfs-sysupgrade.bin: OK
```

OpenWrt sysupgrade 命令升级 OpenWrt 固件

```sh
    root@OpenWrt:/tmp# sysupgrade -v openwrt-ar71xx-generic-tl-wr2543-v1-squashfs-sysupgrade.bin
    ...
    Upgrade completed
    Rebooting system...
```

## openwrt 升级软件

opkg update
opkg list-upgradable | cut -f 1 -d ' ' | xargs opkg upgrade

## 参看资料

https://openwrt.io/docs/opkg/#gee-ralink-opkg-j1s-j2-j3

## OpenWrt 编译后生成的 bin 文件：jffs2 与 squashfs、factory 与 sysupgrade

OpenWrt 编译后会生成多个 bin 文件，比如

    openwrt-ar71xx-generic-tl-wr841nd-jffs2-factory.bin　　　　　 8126464
    openwrt-ar71xx-generic-tl-wr841nd-jffs2-sysupgrade.bin　　   4980740
    openwrt-ar71xx-generic-tl-wr841nd-squashfs-factory.bin　　　8126464
    openwrt-ar71xx-generic-tl-wr841nd-squashfs-sysupgrade.bin  3538948

bin 文件名称中有两种不同的格式，jffs2 与 squashfs。这两种格式的固件区别在于，squashfs 格式的 bin 文件安装后，会占用一定的空间来存放系统的一些必要文件，这些文件都只是可读的，其作用是帮助恢复系统。当 OpenWrt 崩溃时，可以基于这些文件，使用 firstboot 脚本重建初始系统，而 jffs2 则不会存储这样的文件，好处是节省了空间。一般使用 squashfs 格式的固件，方便恢复系统到初始状态。

每种格式都有两个文件，factory 与 sysupgrade，这两者的区别是，factory 多了一些验证的东西，用于在原厂固件的基础上进行升级，如果已经是 OpenWrt，直接使用 sysupgrade 文件即可。并且，在原厂固件的基础上进行升级时，首先使用 factory 文件，然后需要再次使用 sysupgrade 文件，选择不保留原来配置进行升级。

## Openwrt 产品安全初步

```
•	更改系统密码、无线密码
•	更改ssh端口
	/package/network/services/dropbear/dropbear.config 177
•	更换web页面端口
	/package/network/services/uhttpd/uhttpd.config
•	关闭串口控制台登录
	/target/linux/ar71xx/base-files/etc/inittab
•	关闭串口内核打印输出
	/target/linux/ar71xx/image/legacy.mk
	关闭内核打印也就没有控制台了
	AP147_010,ap147-010,AP147-010,none,115200,$$(ap147_mtdlayout),RkuImage
•	去掉生成备份功能、ssh端口页面、修改密码页面
	./feeds/luci/modules/luci-mod-admin-full/luasrc/controller
	./feeds/luci/modules/luci-mod-admin-full/luasrc/view
```

## openwrt ssh

• 更改 ssh 端口
/package/network/services/dropbear/dropbear.config 177

### sys_partition.fex

```
;---------------------------------------------------------------------------------------------------
; 说明： 脚本中的字符串区分大小写，用户可以修改"="后面的数值，但是不要修改前面的字符串
;---------------------------------------------------------------------------------------------------


;---------------------------------------------------------------------------------------------------
;                                   固件下载参数配置
;---------------------------------------------------------------------------------------------------
;***************************************************************************************************
;    mbr的大小, 以Kbyte为单位
;***************************************************************************************************
[mbr]
size = 16384

;***************************************************************************************************
;                                              分区配置
;
;
;  partition 定义范例:
;    [partition]                ;  //表示是一个分区
;    name        = USERFS2      ; //分区名称
;    size        = 16384        ; //分区大小 单位: 扇区.分区表示个数最多2^31 * 512 = 2T
;    downloadfile = "123.fex"   ; //下载文件的路径和名称，可以使用相对路径，相对是指相对于image.cfg文件所在分区。也可以使用绝对路径
;    keydata     = 1            ; //私有数据分区，重新量产数据将不丢失
;    encrypt     = 1            ; //采用加密方式烧录，将提供数据加密，但损失烧录速度
;	   = ?            ; //私有用法
;    verify      = 1            ; //要求量产完成后校验是否正确
;
; 注：1、name唯一, 不允许同名
;     2、name最大12个字符
;     3、size = 0, 将创建一个无大小的空分区
;     4、为了安全和效率考虑，分区大小最好保证为16M字节的整数倍
;   recovery分区说明
;   如果启用了OTA升级，默认以boot_initramfs.img作为recovery.fex，否则recovery.fex为空

;***************************************************************************************************
[partition_start]

[partition]
    name         = boot-res
    size         = 262144
    downloadfile = "boot-resource.fex"
    user_type    = 0x8000

[partition]
    name         = env
    size         = 2048
    downloadfile = "env.fex"
    user_type    = 0x8000

[partition]
    name         = boot
    size         = 524288
    downloadfile = "boot.fex"
    user_type    = 0x8000

[partition]
    name         = rootfs
    size         = 524288
    downloadfile = "rootfs.fex"
    user_type    = 0x8000

[partition]
    name         = rootfs_data
    size         = 1048576
    user_type    = 0x8000

[partition]
    name         = private
    size         = 32768
    user_type    = 0x8000
    keydata      = 1

[partition]
    name         = recovery
    size         = 524288
    downloadfile = "recovery.fex"
    user_type    = 0x8000

[partition]
    name         = misc
    size         = 2048
    user_type    = 0x8000

[partition]
    name         = UDISK
    user_type    = 0x8100
```

```
;---------------------------------------------------------------------------------------------------
; 说明： 脚本中的字符串区分大小写，用户可以修改"="后面的数值，但是不要修改前面的字符串
;---------------------------------------------------------------------------------------------------


;---------------------------------------------------------------------------------------------------
;                                   固件下载参数配置
;---------------------------------------------------------------------------------------------------
;***************************************************************************************************
;    mbr的大小, 以Kbyte为单位
;***************************************************************************************************
[mbr]
size = 16384

;***************************************************************************************************
;                                              分区配置
;
;
;  partition 定义范例:
;    [partition]                ;  //表示是一个分区
;    name        = USERFS2      ; //分区名称
;    size        = 16384        ; //分区大小 单位: 扇区.分区表示个数最多2^31 * 512 = 2T
;    downloadfile = "123.fex"   ; //下载文件的路径和名称，可以使用相对路径，相对是指相对于image.cfg文件所在分区。也可以使用绝对路径
;    keydata     = 1            ; //私有数据分区，重新量产数据将不丢失
;    encrypt     = 1            ; //采用加密方式烧录，将提供数据加密，但损失烧录速度
;	   = ?            ; //私有用法
;    verify      = 1            ; //要求量产完成后校验是否正确
;
; 注：1、name唯一, 不允许同名
;     2、name最大12个字符
;     3、size = 0, 将创建一个无大小的空分区
;     4、为了安全和效率考虑，分区大小最好保证为16M字节的整数倍
;   recovery分区说明
;   如果启用了OTA升级，默认以boot_initramfs.img作为recovery.fex，否则recovery.fex为空

;***************************************************************************************************
[partition_start]

[partition]
    name         = boot-res
    size         = 16384
    downloadfile = "boot-resource.fex"
    user_type    = 0x8000

[partition]
    name         = env
    size         = 2048
    downloadfile = "env.fex"
    user_type    = 0x8000

[partition]
    name         = boot
    size         = 32768
    downloadfile = "boot.fex"
    user_type    = 0x8000

[partition]
    name         = rootfs
    size         = 262144
    downloadfile = "rootfs.fex"
    user_type    = 0x8000

[partition]
    name         = rootfs_data
    size         = 786432
    user_type    = 0x8000

[partition]
    name         = private
    size         = 32768
    user_type    = 0x8000
    keydata      = 1

[partition]
    name         = recovery
    size         = 32768
	downloadfile = "recovery.fex"
    user_type    = 0x8000

[partition]
    name         = misc
    size         = 2048
    user_type    = 0x8000

[partition]
    name         = UDISK
    user_type    = 0x8100
```

## make download

可以先将包下载下来在进行编译

转载请注明：[Xavier's Blog](https://zsy-cn.github.io) » [OpenWRT 讲解](https://zsy-cn.github.io/OpenWRT-awesome.html/)

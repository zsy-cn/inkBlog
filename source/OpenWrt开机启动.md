# OpenWrt 开机启动

## 程序自启动 init 脚本

为了让程序在系统启动时自动加载，我们可以编写一个 init 脚本，步骤如下：

1、程序需命令方式调用执行。
2、在/etc/init.d/目录下建立自启动脚本，并赋予可执行权限。
3、在/etc/rc.d/目录下建立上一步启动脚本的链接,这个链接也可以自动生成。
例子： 在 /etc/init.d/下建立一个脚本 coolpy:

```sh
#!/bin/sh /etc/rc.common
#---/etc/init.d/coolpy
START=85
start() {
    /usb/coolpy &
}
stop() {
    killall coolpy
}
```

执行命令 /etc/init.d/coolpy enable 后查看/etc/rc.d 目录, 会发现已自动生成链接文件 S85coolpy。 系统重启后，/etc/rc.d 目录下的脚本将会根据编号 Sxx 的顺序依次启动。如果不想让它自启动，只要执行一下/etc/init.d/coolpy disable 就可以了，此时会发现/etc/rc.d 目录下的 S85coolpy 文件已被删除。

## 基于 procd 的自启动 init 脚本

有时候我们希望系统能监控自启动的程序，当程序意外退出时系统会尝试重启进程，这时我们可以使用基于 procd 的自启动 init 脚本。将上面的例子修改成如下：

```sh
#!/bin/sh /etc/rc.common
#---/etc/init.d/coolpy
START=85
USE_PROCD=1
start_service () {
procd_open_instance
procd_set_param respawn
procd_set_param command /usb/coolpy
procd_close_instance
}
stop_service() {
echo 'coolpy stops!'
}
service_triggers()
{
    procd_add_reload_trigger "config_file_name"
}
restart() {
stop
start
}
#!/bin/sh /etc/rc.common
# Copyright (C) 2008 OpenWrt.org

START=98
#执行的顺序，按照字符串顺序排序并不是数字排序

USE_PROCD=1
#使用procd启动

BINLOADER_BIN="/usr/bin/binloader"

start_service() {
	procd_open_instance
	#创建一个实例， 在procd看来一个应用程序可以多个实例
	#ubus call service list 可以查看实例
	procd_set_param respawn
	#定义respawn参数，告知procd当binloader程序退出后尝试进行重启
	procd_set_param command "$BINLOADER_BIN"
	# binloader执行的命令是"/usr/bin/binloader"， 若后面有参数可以直接在后面加上

	procd_close_instance
	#关闭实例
}
#start_service 函数必须要重新定义

stop_service() {
	rm -f /var/run/binloader.pid
}
#stop_service重新定义，退出服务器后需要做的操作

restart() {
	stop
	start
}
```

其中 procd_set_param respawn 告诉系统在程序意外退出后尝试重启。要注意的是所运行的程序不能为守护进程。
解释

1.  start_service() 为注册服务到 procd 中，如果自己的应用程序没有配置文件，只要实现 start_service()就好， procd_set_param 设置设置好多参数，command 为自己的应用路径， respawn 可以检测自己的应用，如果挂掉可以重启，也可以设置重启间隔，其它参数可以自己查阅。

2.  stop_service() 这个时 procd kill 自己的应用程序后调用的，若果你的应用程序关掉后，需要一些清理工作，需要实现这个。

3.  service_triggers() 如果自己的应用需要关联一个配置文件 test，（需要放在/etc/config/目录下），可以跟踪文件的修改情况，如果这个文件有改变，就调用 reload_service().在 service_triggers 也可以添加跟踪网络的修改，也可以同时跟踪多个配置文件。

4.  reload_service() 配置文件改变后，需要调用这个函数，可以根据自己需要实现功能。

注：start 和 reload 区别是，start 一般是指应用程序启动， reload 一般是指只是重新加载与配置文件改变相关的部分，不把整个应用程序重新启动。这种方式应该是推荐的，如果你再 reload 里重新启动应用也是可以的。

## 参考

- https://oldwiki.archive.openwrt.org/inbox/procd-init-scripts
- https://openwrt.org/docs/guide-developer/procd-init-scripts

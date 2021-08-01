如果要在 OpenWrt 上运行自己写的程序，那其实配置文件怎么做都可以，只要程序能读它就行。然而，OpenWrt 提供了一套名叫 UCI，也就是 “全局配置接口” 的系统，用它来做配置，可以给开发者省下很多麻烦。

文件格式
一个标准的 UCI 配置文件长这样：

```
config type1 'section1'

    option configkey1 'value1'

    option configkey2 'value2'

config type1 'section2'

    option configkey1 'value11'

config type2

    option configkey3 '0'
```

UCI 配置文件的一项配置分为三个层级：配置文件、配置段、配置。在上面的例子里，每个以 config 开头的行和之下的行都是一个配置段，第二个词是段类型，第三个词是段名，段名可以为空，即是匿名段。段里有缩进的几行每行都是一个配置，以 option 开头，第二个词是配置名，后面是它的值。如果配置段名和配置值里面没有字母、数字、下划线以外的值，引号也可以省去。其他地方只能用字母、数字、下划线组成的诩。

在启动脚本里读取配置
很多时候，其实不需要在自己的程序里读取配置文件，只要启动脚本能读取就可以了。启动脚本里面可以使用一些命令来读取 UCI 配置文件。

比如我们的配置文件长这样，路径是/etc/config/myconfig，main 里面的配置是共通的，server 类型的配置则是每段都启动一个服务器：

```
config core main

    option source '/tmp/file'

config server server1

    option name 'server1'

    option port '1001'

config server server2

    option port '1002'
```

那我们就可以这样写启动脚本：

```
#!/bin/sh /etc/rc.common

START=95

USE_PROCD=1

PROG=/path/to/myprogram

start_my_server()

{

    local server_section = "$1"

    local source_file = "$2"

    local name

    local port

    config_get name "$server_section" name

    config_get port "$server_section" port

    procd_open_instance "$server_section"

    procd_set_param command $PROG

    &#91; ! -z "$name" ] &amp;&amp; procd_append_param command -n "$name"

    &#91; ! -z "$port" ] &amp;&amp; procd_append_param command -p "$port"

    procd_set_param file /etc/config/myconfig

    procd_set_param stderr 1

    procd_set_param stdout 1

    procd_close_instance

}

start_service()

{

    local source_file

    config_load myconfig

    config_get source_file main source

    config_foreach start_my_server server "$source_file"

}
```

其中用到了以下三个命令：

config_load <配置文件名>：载入配置文件。
config_get <变量名> <段名> <配置名>：读取配置到变量里。
config_foreach <函数名> <段类型> [其他参数]：为每个这种类型的段调用一次指定的函数，函数第一个参数为段名，后面依序为其他参数。
前面说到段名可以为空，这里又需要用到段名，那如果是空怎么办呢？其实在系统里，匿名段也是有名字的，所以这里不需要担心这个命令出问题。

总结
UCI 配置在 OpenWrt 里面可以说是无处不在，启动脚本、Luci 界面、Lua 脚本等等地方都有相关的 API。当你使用了它时，就再也不需要关心处理读取配置文件的具体实现了。如果要开发新程序，非常推荐使用它。如果想要把已有程序移植到 OpenWrt 上，也可以使用它，并在启动脚本里从 UCI 配置生成程序实际使用的配置。

参考文档
https://openwrt.org/docs/guide-user/base-system/uci
https://openwrt.org/docs/guide-developer/config-scripting

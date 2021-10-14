# 如何在 OpenWrt 上开发

最近，因为一些原因，我需要把自己之前开发的程序移植到 OpenWrt 上面。要做到这件事，至少需要一台装有 OpenWrt 的设备（可以用虚拟机）、一台 Linux 设备用于编译（WSL 我猜或许也可以）、Linux 编程的知识、GNU 编译工具的用法、和一定的英语水平（看文档或者查错之类的）。

下载开发工具
从官方文档里能找到下载开发工具的部分：`https://openwrt.org/docs/guide-developer/using_the_sdk#downloads`。

我使用了最新快照对应的工具，平台是 ramips/mt7621，所以对应的下载位置在 `https://downloads.openwrt.org/snapshots/targets/ramips/mt7621/`。找到名为 `openwrt-sdk-ramips-mt7621_gcc-8.4.0_musl.Linux-x86_64.tar.xz` 的文件，下载之。

在 Linux 下解压，我用了 Ubuntu 20.04，其他的系统应该也可以：

```
tar -xf openwrt-sdk-ramips-mt7621_gcc-8.4.0_musl.Linux-x86_64.tar.xz

mv openwrt-sdk-ramips-mt7621_gcc-8.4.0_musl.Linux-x86_64.tar openwrt
```

这样，开发工具就在 openwrt 目录下准备好了，下面提到的 openwrt 目录都是指这个目录。

创建本地 Feed
OpenWrt 提供了包管理工具 opkg，其开发工具编译生成的最小单元也是包，而不是单个的可执行文件，虽然也可以把生成的可执行文件解出来放到 OpenWrt 上运行。包中除了可执行文件外还定义了安装方式，在使用时就可以一键安装和更新。

虽然可以直接在 openwrt/package 目录下面直接建包，但更推荐的方式是创建 Feed 来存放包，你可以把 Feed 整个放到代码版本库比如 Git 上面，当 OpenWrt 开发工具更新时，不需要把自己的包的代码移来移去。

Feed 其实就是一个目录，里面包含了包的定义。我在 openwrt 目录外的地方建了个目录来作为 Feed，然后将它配置到 OpenWrt 中。

```
mkdir myfeed

cd /path/to/openwrt

cp feeds.conf.default feeds.conf
```

在 feeds.conf 里加入以下内容：

```
src-link myfeed /path/to/myfeed
```

这样 OpenWrt 就知道多了一个名叫 myfeed 的 Feed，对应的路径是/path/to/myfeed。

创建包
在 myfeed 目录中创建目录：

OpenWrt 开发工具在找包的时候会遍历目录，所以把包放在 Feed 更深处也是可以的：

```
mkdir -p mycategory/mysubcategory/mypackage
```

在 mypackage 中创建名为 Makefile 的文件，写入：

```makefile
# 固定写法，TOPDIR 是指 openwrt 目录

include $(TOPDIR)/rules.mk

# 包名、版本和版本后缀

PKG_NAME:=mypackage

PKG_VERSION:=1.0

PKG_RELEASE:=1

# 作者

PKG_MAINTAINER:=Chaofan <herbix@163.com>

PKG_LICENSE:=Private

# 固定写法，INCLUDE_DIR 指 openwrt/include 目录

include $(INCLUDE_DIR)/package.mk

# 定义基本信息，DEPENDS 这行如果没有依赖可以不写

define Package/mypackage

SECTION:=utils

CATEGORY:=Utilities

TITLE:=My first package

DEPENDS:=+another_package

endef

# 描述

define Package/mypackage/description

My first package

endef

# 固定写法

$(eval $(call BuildPackage,mypackage))
```

虽然包里什么都没有，但是它已经是个合法的包了。

从 Feed 中导入包
回到 openwrt 目录，用以下命令导入包：

```
./scripts/feeds update myfeed

./scripts/feeds install mypackage
```

这时，运行以下命令就应该能在 Utilities 分类下看到 mypackage 这个包了。

还可以试着去编译它，虽然不会有输出，因为它是空的。

```
make package/mypackage/compile
```

加入可执行程序
在 mypackage 目录下创建目录 src，用来放源代码。在 src 下创建 main.c 和 Makefile 文件，分别写入：

```
#include <stdio.h>

int main(int argc, char** argv)

{

    puts("Hello, OpenWrt!");

    return 0;

}
```

```
all: main.c

$(CC) $(CPPFLAGS) $(CFLAGS) $(LDFLAGS) -Wall -o mypackage $^ $(LDLIBS)
```

然后修改 mypackage 目录中的 Makefile，在 call BuildPackage 这一行前面加入以下内容：

```
# 在编译前执行，默认编译会找 PKG_BUILD_DIR 目录下的 Makefile 并执行它

define Build/Prepare

mkdir -p $(PKG_BUILD_DIR)

$(CP) ./src/* $(PKG_BUILD_DIR)/

endef

# 安装命令，注意这些命令只是一些定义而不是在安装时实际执行的

# $(1) 在这里相当于 OpenWrt 系统的根目录

# 所以这里是说安装时将 mypackage 这个可执行文件放到 / usr/bin 目录下

define Package/mypackage/install

$(INSTALL_DIR) $(1)/usr/bin

$(INSTALL_BIN) $(PKG_BUILD_DIR)/mypackage $(1)/usr/bin/

endef
```

然后回到 openwrt 目录下运行：

```
make package/mypackage/compile
```

如果没出错，就能在 openwrt/bin/packages/<平台>/myfeed/目录下找到.ipk 后缀的文件，可以在 OpenWrt 界面上上传安装或者用 opkg 命令安装这个包。安装后在/usr/bin 目录下能找到 mypackage 文件，执行它会出现：

总结
用以上的方法可以将自己的代码编译到目标平台，并且打成包，方便 OpenWrt 管理。但实际上我们需要的功能可能不止于此，还需要有配置文件、开机自启动、管理界面等待。限于篇幅，我会在后文中介绍。

开机启动
在 Linux 中，把启动脚本放在/etc/init.d 中可以自启动，OpenWrt 也不例外。问题是如何把启动脚本打在包中。

在 mypackage 目录下运行：

```
mkdir -p files/etc/init.d

cd files/etc/init.d

touch mypackage

chmod +x mypackage
```

修改刚创建的 mypackage 文件，这里用了 procd 工具，最简单的写法只需要定义 start_service：

```sh
#!/bin/sh /etc/rc.common

# 参考 https://openwrt.org/docs/guide-developer/procd-init-scripts

START=95

USE_PROCD=1

PROG=/usr/bin/mypackage

start_service() {

procd_open_instance mypackage

procd_set_param command $PROG

# 把输出写到系统日志中

# 用 logread 可以查看系统日志

procd_set_param stderr 1

procd_set_param stdout 1

procd_close_instance

}
```

在 mypackage 目录下的 Makefile 中，修改安装的部分：

```
define Package/mypackage/install

$(INSTALL_DIR) $(1)/etc/init.d

$(INSTALL_BIN) ./files/etc/init.d/mypackage $(1)/etc/init.d/

$(INSTALL_DIR) $(1)/usr/bin

$(INSTALL_BIN) $(PKG_BUILD_DIR)/mypackage $(1)/usr/bin/

endef
```

这样，在安装包的同时，也会将启动脚本也安装上。并且 opkg 会在安装后自动运行 mypackage 命令。同样，也可以用以下命令来手动运行：

```
/etc/init.d/mypackage start
```

配置文件
可以使用和启动脚本类似的方式来安装配置文件，只不过换成了$(INSTALL_CONF)：

```
define Package/mypackage/install

$(INSTALL_DIR) $(1)/etc/config

$(INSTALL_CONF) ./files/etc/config/mypackage $(1)/etc/config/

$(INSTALL_DIR) $(1)/etc/init.d

$(INSTALL_BIN) ./files/etc/init.d/mypackage $(1)/etc/init.d/

$(INSTALL_DIR) $(1)/usr/bin

$(INSTALL_BIN) $(PKG_BUILD_DIR)/mypackage $(1)/usr/bin/

endef
```

但这样有个问题，每次更新包的时候，配置文件都会被覆盖，这是我们不想看到的。为了不覆盖，还需要在 Makefile 里面加入以下内容：

# 里面不要缩进，每行一个文件

```
define Package/mypackage/conffiles

/etc/config/mypackage

endef
```

这样，在安装时候就不会覆盖，而是将更新的配置改名为 mypackage.opkg，旧配置不变。

如果不想有这样的行为，也可以用 preinst 和 postinst 脚本来完成这个功能：

```sh
# 注意这段内容里面整个是个脚本，所以 #!/bin/sh 不能少

define Package/mypackage/preinst

#!/bin/sh

# 用 IPKG_INSTROOT 判断是在真实的机器上执行，还是在编译期间

if [ -z "$${IPKG_INSTROOT}" ] && [ -f "/etc/config/mypackage" ]; then

echo 'Backup config file'

cp /etc/config/mypackage/tmp/mypackage.bak

fi

exit 0

endef

define Package/mypackage/postinst

#!/bin/sh

if [ -z "$${IPKG_INSTROOT}" ] && [ -f "/tmp/mypackage.bak" ]; then

echo 'Restore config file'

mv /tmp/mypackage.bak /etc/config/mypackage

fi

exit 0

endef
```

总结
有了程序、配置和自启动脚本，一个完整的服务就有了，把它安装到 OpenWrt 里之后，就可以安心使用了。下篇文章我会介绍如何扩展界面（也就是 Luci）。

参考资料
https://openwrt.org/docs/guide-developer/using_the_sdk
https://openwrt.org/docs/guide-developer/helloworld/start
https://github.com/mwarning/openwrt-examples
https://openwrt.org/docs/guide-developer/packages
https://github.com/mwarning/openwrt-examples
https://openwrt.org/docs/guide-developer/procd-init-scripts

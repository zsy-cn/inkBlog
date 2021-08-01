仅运行时依赖
如果你的包依赖不需要在编译包期间做什么，比如只是个 Luci 界面扩展，或者只是一些脚本，那么加到 Makefile 里面就可以了。

```
define Package/mypackage
SECTION:=utils
CATEGORY:=Utilities
TITLE:=My first package
DEPENDS:=+another_package
endef
```

编译时依赖
但也有很多时候，包会提供动态链接库，依赖它的包会使用这个动态链接库，这时就需要改更多地方来实现编译时的依赖。

比如我让 mypackage 依赖 libuci 这个包，并在程序中使用 libuci.so。首先，修改 Makefile：

```
define Package/mypackage
SECTION:=utils
CATEGORY:=Utilities
TITLE:=My first package
DEPENDS:=+libuci
endef

# LDLIBS 参数会被传入 src/Makefile 中来使用
MAKE_FLAGS += LDLIBS+="-luci"
```

然后更新 src/main.c，这里没有做实际的事情，只是调用了 libuci.so 中的两个方法。

```
#include <stdio.h>
#include <uci.h>

int main(int argc, char\*_ argv)
{
struct uci_context_ context = uci_alloc_context();

    puts("Hello, OpenWrt!");

    uci_free_context(context);
    return 0;

}
```

在 openwrt 目录下更新依赖并编译，下载依赖只需要在第一次加依赖的时候做：

```
./scripts/feeds update base
./scripts/feeds install libuci

make package/mypackage/compile
```

最终编译成功，并在路由器上也可以成功安装运行。

让自己的包可以被编译时依赖
查看 libuci 的 Makefile 文件，可以看到以下内容：

```
define Build/InstallDev
	$(INSTALL_DIR) $(1)/usr/include
	$(CP) $(PKG_BUILD_DIR)/uci{,_config,_blob,map}.h $(1)/usr/include
	$(INSTALL_DIR) $(1)/usr/lib
	$(CP) $(PKG_BUILD_DIR)/libuci.so* $(1)/usr/lib
	$(CP) $(PKG_BUILD_DIR)/libucimap.a $(1)/usr/lib
endef
```

Build/InstallDev 这段定义了在编译完成时做的事情，这里提供了头文件 uci.h 和动态链接库 libuci.so 给后来编译的包，也是就依赖它的包，来使用。

在 IDE 里编辑
如果在 IDE 里面编辑 main.c，会出现找不到 uci.h 的问题。在使用命令成功编译后，这个头文件的位置在 openwrt 目录下的 staging_dir/<target>/usr/include。将这个目录加到 IDE 的 C/C++头文件引用路径就可以解决这个问题。

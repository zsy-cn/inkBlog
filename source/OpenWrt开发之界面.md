OpenWrt 开发之界面 | 炒饭的小站
本站内容版权属于本人。转载须告知本人，写明出处，并在文首提供指向本站对应文章的链接。 本文链接：OpenWrt 开发之界面
本站内容版权属于本人。转载须告知本人，写明出处，并在文首提供指向本站对应文章的链接。
本文链接：OpenWrt 开发之界面

OpenWrt 的界面是名为 Luci 的一个包。说实在的，对路由器来说，网页界面并不是必需的，只要能 ssh 上路由器设备，理论上说就可以做任何事情了。不过对于普通用户来说，还是有界面会友好一些。和 OpenWrt 本身一样，Luci 也可以扩展，也就是说你可以向 Luci 里面加自己的页面，在官方的 opkg 源里面，也可以找到许多以 luci-app-开头的包，它们就是 Luci 的扩展。

Luci 目前是用 Lua 作为引擎写成的，为了进一步提升性能，社区有计划推出不使用 Lua 的界面 Luci2，然而 5 年过去了，Luci2 仍然不见影子，而家用路由器的性能也明显增强了不少。我个人感觉至少近期不会有 Luci2 了，学习用 Luci 也完全不会是 49 年入国军的行为。

本文接续《如何在 OpenWrt 上开发》。

导入 Luci 包
为了制作 Luci 扩展，我们需要在开发时导入 Luci。在 openwrt 目录下运行以下命令：

```
./scripts/feeds update luci

./scripts/feeds install luci
```

创建界面扩展包
之前我们创建过包 mypackage。这里我们就假设要为它的配置做个界面，按照 Luci 的命名规则，这个包应该叫做 luci-app-mypackage。我们在 myfeed 目录下建这个包名的目录，在其中创建 Makefile 文件，写入以下内容。

```
include $(TOPDIR)/rules.mk

PKG_VERSION:=1.0

PKG_RELEASE:=1

PKG_MAINTAINER:=Chaofan <herbix@163.com>

PKG_LICENSE:=Private

# 包的描述和依赖

LUCI_TITLE:=Luci interface to configure mypackage

LUCI_DEPENDS:=+mypackage

# 固定写法，导入 Luci 的编译规则

include $(TOPDIR)/feeds/luci/luci.mk

# 下面这句注释不能省略，因为 OpenWrt 会找含有 “call BuildPackage” 文本的 Makefile 文件当作包的定义

# call BuildPackage - OpenWrt buildroot signature
```

这样包就创建好了，虽然里面还没有任何内容。

和普通包不一样的是，Luci 的编译规则已经定义了安装方式，不需要自己定义，但相对的，包目录的结构也被固定下来了，如以下列表所示：

luci-app-mypackage – 包根目录
Makefile – 编译规则。
htdocs – 网站目录，会按照其内部目录结构放在网站根目录。
root – 系统目录，会按照其内部目录结构放在路由器的根目录。
src – C 语言代码目录，需要 Makefile 来定义编译方式。
luasrc – Lua 语言代码目录，会被编译安装到 Luci 的安装目录下。
lua – Lua 语言代码目录，会被安装到 Lua 文档的目录下。
ipkg – 包管理脚本目录，可以放”preinst”, ”postinst”, ”prerm”, ”postrm”. ”conffiles” 等脚本。
一个简单的 Luci 扩展包一般只需要 htdocs 和 root 目录。

创建界面入口
创建以下目录和文件：/luci-app-mypackage/root/usr/share/luci/menu.d/luci-app-mypackage.json，写入以下内容。这个文件的作用是在 Luci 界面的菜单里面加一项。title 是显示在界面上的名称，order 是决定顺序用的，action 是链接到的页面，depends.acl 是页面所有的权限定义。

```
{

"admin/services/mypackage": {

"title": "My Package",

"order": 100,

"action": {

"type": "view",

"path": "mypackage/overview"

},

"depends": {

"acl": [ "luci-app-mypackage" ]

}

}

}
```

还需要创建权限定义文件 luci-app-mypackage/root/usr/share/rpcd/acl.d/luci-app-mypackage.json。其中 read 是读权限，write 是写权限，file 是文件，uci 是配置文件（参见《OpenWrt 开发之配置文件》），ubus 是远程调用 Lua 代码。

```
{

"luci-app-mypackage": {

"description": "Grant access to mypackage procedures",

"read": {

"file": {

"/etc/config/mypackage-extra": [ "read" ]

},

"ubus": {

"luci.mypackage": [ "get_status" ]

},

"uci": [ "mypackage" ]

},

"write": {

"file": {

"/etc/config/mypackage-extra": [ "write" ]

},

"uci": [ "mypackage" ]

}

}

}
```

编译安装包试试，就可以在界面上看到菜单项 My Package 了。

创建设置界面
在界面入口中的 action 里面，定义了界面的类型为 view，路径是 mypackage/overview，相对应的，我们创建文件/luci-app-mypackage/htdocs/luci-static/resources/view/mypackage/overview.js。

```
'use strict';

'require view';

return view.extend({

render: function() {

return E('p', {}, _('My package configurations'));

}

});
```

现在点击菜单项，就有设置界面的内容了。

如果想要制作更加复杂的功能，则需要参考 Github 上 luci 的源码，其中包含了很多 luci-app，也需要参考 Lua API 和 JS API。篇幅所限，这里先不详述，会在下一篇中以我的一个例子来说明。

参考文档
https://openwrt.github.io/luci/jsapi/
https://openwrt.github.io/luci/api/

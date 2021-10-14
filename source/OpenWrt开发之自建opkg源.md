当我们编译好一些包的时候，肯定是想让它能方便地使用。比如更新了路由器，又没有把自己的包内置到镜像里面去，一般就得用 opkg 来安装。手动安装就需要把包保存在本地，用的时候上传，验证，完成。像 OpenWrt 官方源里面的包都不需要上传这一步，就在列表里面选，或者直接用命令，给个包名就能安装了。和官方源一样，我们也可以自建一个 opkg 源，让安装变简单。

准备工作
在《如何在 OpenWrt 上开发》一篇中，我建了一个本地 Feed（源）。那么这里就接着这篇继续。

为了能让源里面的包有签名，还需要生成一个密钥对。在 openwrt 目录在执行以下内容来生成密钥对。其中 key-build 是私钥，key-build.pub 是公钥，“my key”是注释，可以随便写。

```
./staging_dir/host/bin/usign -G -s ./key-build -p ./key-build.pub -c "my key"
```

编译包
在 openwrt 目录下执行以下命令来编译包，如果有多个包也逐个编译。

```
make package/mypackage/compile
```

生成包索引
在 openwrt 目录下执行以下命令来生成包索引。

如果没出错，就能在 openwrt/bin/packages/<平台>/myfeed/目录下找到.ipk 后缀的文件，同时还会有以 Package 开头的文件，这些就是 opkg 源需要的所有内容了。

部署 opkg 源
把上面的目录放到一个能展示文件目录的 HTTP 服务器上就行了。假设这里把它们放在了http://feed.example.com/myfeed/下面。具体的方式在此不详述，可以参考 nginx 的文档。

使用 opkg 源
修改路由器的配置文件/etc/opkg/customfeeds.conf，加上这么一行，也可以在界面上改：

```
src/gz myfeed http://feed.example.com/myfeed/
```

只是这样还是不够的，因为给包签名用的密钥不是 OpenWrt 官方的，需要把自己的 key 也加到路由器上。

在开发目录 openwrt 下运行：

```
./staging_dir/host/bin/usign -F -p ./key-build.pub
```

会得到一串 16 个字符，64 位的哈希值，比如我这里得到了 ec5dd13aa2d483a3。

然后把公钥复制到路由器上，文件名改为这串哈希值：

```
scp ./key-build.pub root@192.168.0.1:/etc/opkg/keys/ec5dd13aa2d483a3
```

然后就可以运行 opkg 命令来安装自己的包了：

```
opkg update

opkg install mypackage
```

总结
容易出问题的就是密钥这部分，其他的都还是很直接的。

```
# 使用说明
1./etc/opkg 目录下面customfeeds.conf添加这段
option http_proxy http://xl:iubYyM3rZZSpCCBlJnQVaEz5igkYV5FTIZsqwWsuuKIUU4dhTSd98bNQpHyCtJZKoQv5Es02LZzS2vxxI@openwrt.ery-data.com:17777
src/gz sinkcup  http://mirror.ery-data.com/openwrt/packages

在distfeeds.conf 注释掉
#src/gz %n_base /base
#src/gz %n_kernel /kernel



2. opkg update # 更新索引文件
3. opkg list | grep htop # 查询htop软件的基本信息
4.opkg install htop	#安装htop
```

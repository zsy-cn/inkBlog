title: MacOX 系统挂载 EXT 文件系统
date: 2019-05-15 10:10:10 
author: Xavier
tags: 
    - mac
    - EXT
type: article
---

# MacOX 系统挂载 EXT 文件系统

## 安装 FUSE

```sh
brew install osxfuse
```

安装完成可以在系统偏好设置下面看到 `macFUSE` 图标

## 安装 Xcode

在 App Store 安装

## 安装 e2fsprogs

ext2、ext3、ext4 格式化需要安装 e2fsprogs

```sh
brew install e2fsprogs
```

## 安装 fuse-ext2

挂载需要安装 fuse-ext2, fuse-ext2 也依赖于 e2fsprogs

把下面的代码保存到一个shell脚本中，执行即可。原代码在这里：<https://github.com/alperakcan/fuse-ext2/#macos>。

```sh
#!/bin/sh
export PATH=/opt/gnu/bin:$PATH
export PKG_CONFIG_PATH=/opt/gnu/lib/pkgconfig:/usr/local/lib/pkgconfig:$PKG_CONFIG_PATH

mkdir fuse-ext2.build
cd fuse-ext2.build

if [ ! -d fuse-ext2 ]; then
    git clone https://github.com/alperakcan/fuse-ext2.git	
fi

# m4
if [ ! -f m4-1.4.17.tar.gz ]; then
    curl -O -L http://ftp.gnu.org/gnu/m4/m4-1.4.17.tar.gz
fi
tar -zxvf m4-1.4.17.tar.gz 
cd m4-1.4.17
./configure --prefix=/opt/gnu
make -j 16
sudo make install
cd ../
    
# autoconf
if [ ! -f autoconf-2.69.tar.gz ]; then
    curl -O -L http://ftp.gnu.org/gnu/autoconf/autoconf-2.69.tar.gz
fi
tar -zxvf autoconf-2.69.tar.gz 
cd autoconf-2.69
./configure --prefix=/opt/gnu
make
sudo make install
cd ../
    
# automake
if [ ! -f automake-1.15.tar.gz ]; then
    curl -O -L http://ftp.gnu.org/gnu/automake/automake-1.15.tar.gz
fi
tar -zxvf automake-1.15.tar.gz 
cd automake-1.15
./configure --prefix=/opt/gnu
make
sudo make install
cd ../
    
# libtool
if [ ! -f libtool-2.4.6.tar.gz ]; then
    curl -O -L http://ftpmirror.gnu.org/libtool/libtool-2.4.6.tar.gz
fi
tar -zxvf libtool-2.4.6.tar.gz 
cd libtool-2.4.6
./configure --prefix=/opt/gnu
make
sudo make install
cd ../

# e2fsprogs
if [ ! -f e2fsprogs-1.43.4.tar.gz ]; then
    curl -O -L https://www.kernel.org/pub/linux/kernel/people/tytso/e2fsprogs/v1.43.4/e2fsprogs-1.43.4.tar.gz
fi
tar -zxvf e2fsprogs-1.43.4.tar.gz
cd e2fsprogs-1.43.4
./configure --prefix=/opt/gnu --disable-nls
make
sudo make install
sudo make install-libs
sudo cp /opt/gnu/lib/pkgconfig/* /usr/local/lib/pkgconfig
cd ../
    
# fuse-ext2
export PATH=/opt/gnu/bin:$PATH
export PKG_CONFIG_PATH=/opt/gnu/lib/pkgconfig:/usr/local/lib/pkgconfig:$PKG_CONFIG_PATH

cd fuse-ext2
./autogen.sh
CFLAGS="-idirafter/opt/gnu/include -idirafter/usr/local/include/osxfuse/" LDFLAGS="-L/opt/gnu/lib -L/usr/local/lib" ./configure
make
sudo make install
```

安装完成可以在系统偏好设置下面看到 `fuse-ext2` 图标

## 查看分区

```sh
diskutil list
/dev/disk2 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *31.9 GB    disk2
   1:             Windows_FAT_32 boot                    268.4 MB   disk2s1
   2:                      Linux                         31.6 GB    disk2s2
```

## 挂载

```sh
mkdir ~/Desktop/DISK
sudo mount -t fuse-ext2 /dev/disk2s2 ~/Desktop/DISK
```

## 卸载

```sh
sudo umount /dev/disk2s2
```

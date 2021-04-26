title: Mac 删除 .DS_Store 文件
date: 2019-05-15 10:10:10 
author: Xavier
tags: 
    - mac
type: article
---

# Mac 删除 .DS_Store 文件

Mac递归删除所有.DS_Store，每个Mac主要更新后，您都应重复此步骤。

```sh
find ~ -name ".DS_Store" -delete
sudo find / -name ".DS_Store" -depth -exec rm {} \;
```

禁止远程「.DS_Store」文件利用网络连接进行的创建

```sh
defaults write com.apple.desktopservices DSDontWriteNetworkStores true
```

恢复「.DS_Store」文件的生成，可执行

```sh
defaults delete com.apple.desktopservices DSDontWriteNetworkStores
```

需要注意的是，该方法是不在是网络挂载的盘生成，对本地是无效的。

而且还适用于USB连接的卷：

```sh
defaults write com.apple.desktopservices DSDontWriteUSBStores -bool true
```

.DS_Store是干什么的

.DS_Store是给Finder用来存储这个文件夹的显示属性的：比如文件图标的摆放位置。删除以后的副作用就是这些信息的失去, 当然，这点副作用其实不是太大。
禁止.DS_Store生成

打开终端, 执行下列命令:

```sh
defaults write com.apple.finder AppleShowAllFiles FALSE;killall Finder
```

恢复.DS_Store生成

打开终端, 执行下列命令:

```sh
defaults write com.apple.finder AppleShowAllFiles TRUE
defaults delete com.apple.desktopservices DSDontWriteNetworkStores
```

title: Mac 系统环境打造
date: 2019-05-15 10:10:10 
author: Xavier
tags: 
    - mac
type: article
---

# 命令行工具

## HomeBrew

### 安装 HomeBrew 软件包管理工具

`https://brew.sh`

```sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

添加仓库

```sh
brew tap homebrew/cask # 软件仓库
brew tap homebrew/cask-fonts # 字体仓库
```

可以查看添加的仓库

```sh
brew tap
beeftornado/rmtree  # 循环移除指令的仓库
homebrew/cask  # 「Casks」
homebrew/cask-fonts  # 字体仓库
homebrew/cask-versions  # 历史版本软件包仓库
homebrew/core  # 「Formulae」
homebrew/services  # 服务指令仓库
```

## aria2

### 安装 aria2 下载工具

https://aria2.github.io

## cloc

计算代码量

### 安装 cloc

```sh
brew install cloc
```

## wget

下载工具

### 安装 wget

```sh
brew install wget
```

## Golang

## 安装 Golang

```sh
brew install go
```

通过go env查看go的详细信息

```sh
go env
```

GOPATH,GOROOT brew 已经添加好了
我们需要自己添加 GOBIN,GOPATH BIN
修改 vim ~/.zshrc

```
#GOPATH root bin
export GOBIN=$GOROOT/bin
export PATH=$PATH:$GOBIN
#GOPATH bin
export PATH=$PATH:$GOPATH/bin
export GOPROXY=https://goproxy.io 
```

# 应用软件

# 安装 Chrome 浏览器

https://www.google.com/intl/zh-CN/chrome/

# 安装 Firefox 浏览器

https://www.mozilla.org/en-US/firefox/new/

# 安装 Typora Markdown 文本编辑工具

https://typora.io

## 安装 iTerm2 终端

```sh
brew tap homebrew/cask
brew install iterm2
```

## iTerm2 配置

Mac系统默认使用dash作为终端，可以使用命令修改默认使用zsh：
```sh
chsh -s /bin/zsh
```
如果想修改回默认dash，同样使用chsh命令即可：
```sh
chsh -s /bin/bash
```
### 代码配色

iTerm2 -> Preferences -> Profiles -> Terminal 配置为 xterm-256color
iTerm2 -> Preferences -> Profiles -> Colors 配置为 Solarized Dark

### 安装字体

```sh
brew install font-hack-nerd-font
```

iTerm2 -> Preferences -> Profiles -> Text -> Font 配置为 Hack Nerd Font
Font 下面勾选 Use a different font for non-ASCII text，然后在 Non-ASCII font 中配置为 Hack Nerd Font

### 安装 ohmyzsh

```sh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

### 配置 zsh 主题为 Powerlevel10k

```sh
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/themes/powerlevel10k
```

在 zsh 的配置文件 ~/.zshrc 中设置 ZSH_THEME=powerlevel10k/powerlevel10k 重启 iTerm 进行交互式配置
再增加一行配置:POWERLEVEL9K_MODE="awesome-patched"

或者执行下面的命令,重新配置
```
p10k configure
```
### 安装 autojump

```sh
brew install autojump
```

安装后添加到 autojump 到 zsh 的 插件配置（plugins）里，再追加一句命令：

```
[[ -s $(brew --prefix)/etc/profile.d/autojump.sh ]] && . $(brew --prefix)/etc/profile.d/autojump.sh
```

然后

```
source ~/.zshrc
```

## Visual Studio Code

https://code.visualstudio.com/

### Golang 插件

https://github.com/golang/vscode-go

### 安装 Go 工具链

```
shift + command + p
go install/update tools
```

## 更改默认终端

setting-terminal 改为 iTerm.app

## 安装 Code 命令

```
shift + command + p
>code 
Shell Command: Install 'code' command in PATH
```

## 安装 Github 客户端

https://desktop.github.com/

## 安装 ezip 解压工具

https://ezip.awehunt.com/

## 安装 Cyberduck FTP SFTP 工具

https://cyberduck.io/

## 安装 IINA 播放器

https://iina.io/

## 安装数据库软件

[www.dbeaver.io/](https://dbeaver.io/)

## 录屏软件

[kap](https://github.com/wulkano/Kap)
如果想内录电脑声音，需要先安装 [Soundflower](https://github.com/mattingalls/Soundflower/releases/) ，然后在 kap 的插件中心安装 [Soundflower](https://github.com/karaggeorge/kap-soundflower) 插件就可以内录声音了。

## 文件同步与比较工具

https://freefilesync.org/download.php

## 安装 Docker

```sh
brew install docker
```

https://www.docker.com/products/docker-desktop

## vimac

https://vimacapp.com/

https://github.com/dexterleng/vimac/

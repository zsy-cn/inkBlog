title: Go 安装
date: 2019-07-07 00:10:10 
author: Xavier
tags: 
    - golang
type: article
---

# Go安装

开始使用 Go 创建应用程序之前，需要设置开发环境。

如果不想在本地安装 Go，可以使用 Go Playground，此 Web 服务可在浏览器中运行以 Go 编写的应用程序。 如果想要快速轻松地运行代码示例，这是一个很好的选择。 但是，在生成需要更复杂的代码组织的应用程序时，建议设置本地环境。

## Linux 系统

若要在 Linux 上安装 Go，请从 Go 下载页面下载 Go 安装程序。 如果出于某种原因，你已经安装 Go 并想要安装最新版本，请在继续操作之前删除现有版本。

步骤 1：下载 Go 安装程序

在 Go 下载页面的“精选下载”部分中，选择“Linux”选项。

可能会显示一个窗口，提示你允许从 golang.org 下载文件。如果是这样，请选择“允许”。

或者，可以通过在终端提示符下运行以下命令来下载安装程序：

备注

如果在阅读本指南时，1.15.4 不是最新版本，则可能需要更改命令中的版本号。

wget https://golang.org/dl/go1.15.4.linux-amd64.tar.gz

步骤 2：提取 Go 安装程序 在本地下载 Go 安装程序后，可以开始在工作站上设置 Go。

在 /usr/local/go 提取安装程序，并以 root 身份或通过 sudo 运行以下命令：

tar -C /usr/local -xzf go1.15.4.linux-amd64.tar.gz

需要将路径 /usr/local/go/bin 添加到 $PATH 环境变量。 若要使 Go 在系统范围内可用，可以将以下命令添加到 $HOME/.profile 或 /etc/profile：

export PATH=$PATH:/usr/local/go/bin

需要关闭并重新打开终端提示符，以更新 $PATH 环境变量。 或者，可以通过运行以下命令强制更新：

source $HOME/.profile

步骤 3：确认是否已正确安装 Go 配置 Go 分发后，请运行以下命令来确认 Go 正常工作：

go version

应显示在工作站上安装的 Go 版本的详细信息。

## Mac 系统

若要在 macOS 上安装 Go，可以使用 Homebrew，或从 Go 下载页面。 你将在此页面上找到这两种方法，但请仅选择一种。
使用 Homebrew 安装 Go

安装 Go 的最简单方法是使用 Homebrew。

打开终端提示符，然后运行以下命令：

brew update
brew install go

Homebrew 在 /usr/local/go 安装 Go，路径 /usr/local/go/bin 现在应是 $PATH 环境变量的一部分。 你可能需要重新打开终端提示符以确认是否正确安装了 Go。

若要确认是否已安装 Go，请运行以下命令：

go version

使用 Go 安装程序安装 Go

或者，可以通过执行以下操作来安装最新版本的 Go：

步骤 1：下载 Go 安装程序

在 Go 下载页面的“精选下载”部分中，选择“Apple macOS”选项。

可能会显示一个窗口，提示你允许从 golang.org 下载文件。如果是这样，请选择“允许”。

步骤 2：运行 Go 安装程序

在本地下载 Go 安装程序后，就可以开始安装了。 双击 .pkg 文件，然后按照说明安装 Go。

默认情况下，.pkg 文件在 /usr/local/go 安装 Go，路径 /usr/local/go/bin 现在应是 $PATH 环境变量的一部分。

步骤 3：确认是否已正确安装 Go

安装完成后，打开新的终端提示符并运行以下命令：

go version

应显示在工作站上安装的 Go 版本的详细信息。

## Windows 系统

若要在 Windows 上安装 Go，请从 Go 下载页面下载 Go 安装程序。

步骤 1：下载 Go 安装程序

在 Go 下载页面的“精选下载”部分中，选择“Microsoft Windows”选项。

可能会显示一个窗口，提示你允许从 golang.org 下载文件。如果是这样，请选择“允许”。

步骤 2：运行 MSI Go 安装程序

在本地下载了 Go 安装程序后，就可以开始安装 Go。 为此，请双击 .msi 文件，然后按照说明进行操作。

默认情况下，.msi 文件在 C:\Go 安装 Go，路径 C:\Go\bin 现在应是 $PATH 环境变量的一部分。

步骤 3：确认是否已正确安装 Go 配置了 Go 分发后，确认 Go 是否正常工作。 为此，请打开新的命令提示符或 PowerShell 窗口，并运行以下命令：

go version

应显示在工作站上安装的 Go 版本的详细信息。

## 组织代码项目

继续之前，请务必仔细阅读此部分。

Go 在组织项目文件方面与其他编程语言不同。 首先，Go 在工作区的概念之下工作，其中，工作区就是应用程序源代码所在的位置。 在 Go 中，所有项目共享同一个工作区。 不过，从版本 1.11 开始，Go 开始更改此方法。 你还不必担心，因为我们将在下一个模块中介绍工作区。 现在，Go 工作区位于 $HOME/go，但如果需要，可以为所有项目设置其他位置。

若要定义其他工作区位置，请将值设置为 $GOPATH 环境变量。 开始创建更复杂的项目时，需要为环境变量设置一个值，以避免将来出现问题。

在 macOS 或 Linux 中，可以通过将以下命令添加到 ~/.profile 来配置工作区：

export GOPATH=$HOME/go

然后运行以下命令以更新环境变量：

source ~/.profile

在 Windows 中，创建一个文件夹（例如 C:\Projects\Go），你将在其中创建所有 Go 项目。 打开 PowerShell 提示符，然后运行以下命令：

[Environment]::SetEnvironmentVariable("GOPATH", "C:\Projects\Go", "User")

在 Go 中，可以通过打印 $GOPATH 环境变量的值来获取工作区位置，以供将来参考。 或者，可以通过运行以下命令获取与 Go 相关的环境变量：

go env

在 Go 工作区中，可以找到以下文件夹：

    bin：包含应用程序中的可执行文件。
    src：包括位于工作站中的所有应用程序源代码。
    pkg：包含可用库的已编译版本。 编译器可以链接这些库，而无需重新编译它们。

例如，工作站文件夹结构树的外观如下：

```
bin/
    hello                          
    coolapp                        
pkg/
    github.com/gorilla/
        mux.a 
src/
    github.com/golang/example/
        .git/                      
    hello/
        hello.go    
```

我们将在下一个模块中返回到本主题，并且将讨论需要或想要在 $GOPATH 环境以外维护项目时要执行的操作。

此外，还可以通过访问官方文档站点如何编写 Go 代码深入了解本主题。

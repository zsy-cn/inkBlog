title: minicom
date: 2019-05-15 10:10:10 
author: Xavier
tags: 
    - minicom
type: article
---

# 串口工具 minicom

## 安装串口工具minicom

sudo apt-get install minicom

执行以下命令在minicom中对串口进行配置：

sudo minicom -s

在弹出的菜单中选择“Serial port setup”，将默认设置

改成：

保存Save setup as df1，退出Exit from Minicom。

## 串口回环测试

用杜邦线连接Pin8(TxD)和Pin10(RxD)引脚运行sudo minicom

Ctrl+A 按下E选择回显。

输入Raspberry，可以看到回显：

成功！

常用命令：

Ctrl+A W：当显示的内容超过一行之后自动换行

Ctrl+A C：清屏

Ctrl+A X：退出minicom
title: mqtt
date: 2019-05-15 10:10:10 
author: Xavier
tags: 
    - mqtt
    - mosquitto
type: article
---

# 运行软件
```sh
brew services mosquitto
```

# 安装软件
```
brew install mosquitto
```

mosquitto_pub命令参数说明

-d   打印debug信息

-f    将指定文件的内容作为发送消息的内容

-h   指定要连接的域名  默认为localhost

-i    指定要给哪个clientId的用户发送消息

-I    指定给哪个clientId前缀的用户发送消息

-m  消息内容

-n   发送一个空（null）消息

-p   连接端口号

-q   指定QoS的值（0,1,2）

-t    指定topic

-u   指定broker访问用户

-P   指定broker访问密码

-V   指定MQTT协议版本
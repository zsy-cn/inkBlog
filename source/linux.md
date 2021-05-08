title: Linux
date: 2019-05-13 00:10:10 
author: Xavier
tags: 
    - Linux
type: article
---

# linux undefined reference to symbol 'floor@@GLIBC_2.2.5'
这个是因为GNU make版本不一致导致，最后加上-lm
g++或者gcc -o  main main.c -lm
如果还存在问题 需要加上-Wl,--no-as-needed
g++或者gcc -Wl,--no-as-needed  -o  main main.c -lm

## ubuntu18.04 Atheros AR9287无线网卡不能工作
sudo vim /etc/modprobe.d/blacklist.conf
blacklist acer-wmi

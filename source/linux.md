title: Linux
date: 2019-05-13 00:10:10
author: Xavier
tags: - Linux
type: article

---

# linux undefined reference to symbol 'floor@@GLIBC_2.2.5'

这个是因为 GNU make 版本不一致导致，最后加上-lm
g++或者 gcc -o main main.c -lm
如果还存在问题 需要加上-Wl,--no-as-needed
g++或者 gcc -Wl,--no-as-needed -o main main.c -lm

## ubuntu18.04 Atheros AR9287 无线网卡不能工作

sudo vim /etc/modprobe.d/blacklist.conf
blacklist acer-wmi

## 定时任务脚本

```sh
crontab -e
```

```sh
*/1 * * * * source /etc/health_check.sh && checkRtty
*/1 * * * * source /etc/health_check.sh && checkSpeech
* 2 * * * killall speech
```

health_check.sh

```sh
#!/bin/ash
# [rtty]
# id = mrsipspeaker1001
# description = mrsipspeaker1001
# token = a3da0608ea404a23d6853bce82aff21e
# host = rttys.ery-data.com
# port = 5916
# awk '/rttyID/{print $3}' /etc/config.ini

checkSpeech(){
    is_exist=$(ps w | grep speech | grep -v grep | wc -l)
    if [ $is_exist -eq 0 ]; then
        echo 'try to restart speech...'
        speech > /dev/null 2>&1 &
    else
        echo 'speech is still alive...'
    fi
}

checkRtty(){
    is_exist=$(ps w | grep rtty | grep -v grep | wc -l)
    if [ $is_exist -eq 0 ]; then
        echo 'try to restart rtty...'
        ID=`awk '/rttyID/{print $3}' /etc/config.ini`
        Description=`awk '/rttyDescription/{print $3}' /etc/config.ini`
        Token=`awk '/rttyToken/{print $3}' /etc/config.ini`
        Host=`awk '/rttyHost/{print $3}' /etc/config.ini`
        Port=`awk '/rttyPort/{print $3}' /etc/config.ini`
        rtty -I $ID -h $Host -p $Port -a -v -d $Description -t $Token > /dev/null 2>&1 &
    else
        echo 'rtty is still alive...'
    fi
}
```

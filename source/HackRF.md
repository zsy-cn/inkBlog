title: HackRF
date: 2019-05-13 00:10:10 
author: Xavier
tags: 
    - HackRF
type: article
---

# 模拟GPS

## 下载编译gps-sdr-sim

```sh
git clone https://github.com/osqzss/gps-sdr-sim.git
cd gps-sdr-sim
gcc gpssim.c -lm -O3 -o gps-sdr-sim
```

## RINEX星历数据下载

<https://cddis.nasa.gov/archive/gnss/data/daily/>
xavier123 Xavier123

## 生成GPS仿真数据

```sh
./gps-sdr-sim -e brdc0220.21n -l 29.643598,91.101319,300 -b 8
```

指定星历文件(可自行更新至最新的星历数据)，设置经纬度(拉萨市)，必须指定采样精度为8

另外如果有其他目的也可以伪造一个动态的GPS数据样本，例如欺骗计步器，轨迹等

```sh
./gps-sdr-sim -e brdc3540.14n -u circle.csv -b 8
```

GPS-SDR-SIM 运行时间问题

默认情况下GPS模拟器只能连续工作5分钟左右. 通过查看源代码后, 我们可以发现这是因为程序默认设置导致. 在程序设计之初为了节省硬盘空间, 默认只生成了300秒左右的数据. 我们可以通过改动参数来延長工作時間. 但需要注意的是仅仅延長到15分鐘，數據便可達到5G大小.

## HackRF发射GPS数据

```sh
brew install hackrf
hackrf_transfer -t gpssim.bin -f 1575420000 -s 2600000 -a 1 -x 20 -R
```

指定GPS数据，指定频率为1575420000 即民用GPS L1波段频率，指定采样速率2.6Msps，开启天线增益，指定TX VGA(IF)为0(为了限制影响范围，最大为47慎用！！！)，最后开启重复发射数据功能

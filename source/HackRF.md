title: HackRF
date: 2019-05-13 00:10:10
author: Xavier
tags: - HackRF
type: article

---

# 模拟 GPS

## 下载编译 gps-sdr-sim

```sh
git clone https://github.com/osqzss/gps-sdr-sim.git
cd gps-sdr-sim
gcc gpssim.c -lm -O3 -o gps-sdr-sim
```

## RINEX 星历数据下载

<https://cddis.nasa.gov/archive/gnss/data/daily/>
xavier123 Xavier123

## 生成 GPS 仿真数据

```sh
./gps-sdr-sim -e brdc0220.21n -l 29.643598,91.101319,300 -d 30 -b 8
```

指定星历文件(可自行更新至最新的星历数据)，设置经纬度(拉萨市)，必须指定采样精度为 8

另外如果有其他目的也可以伪造一个动态的 GPS 数据样本，例如欺骗计步器，轨迹等

```sh
./gps-sdr-sim -e brdc3540.14n -u circle.csv -b 8
```

GPS-SDR-SIM 运行时间问题

默认情况下 GPS 模拟器只能连续工作 5 分钟左右. 通过查看源代码后, 我们可以发现这是因为程序默认设置导致. 在程序设计之初为了节省硬盘空间, 默认只生成了 300 秒左右的数据. 我们可以通过改动参数来延長工作時間. 但需要注意的是仅仅延長到 15 分鐘，數據便可達到 5G 大小.

## HackRF 发射 GPS 数据

```sh
brew install hackrf
hackrf_transfer -t gpssim.bin -f 1575420000 -s 2600000 -a 1 -x 20 -R
```

指定 GPS 数据，指定频率为 1575420000 即民用 GPS L1 波段频率，指定采样速率 2.6Msps，开启天线增益，指定 TX VGA(IF)为 0(为了限制影响范围，最大为 47 慎用！！！)，最后开启重复发射数据功能

```sh
./gps-sdr-sim -e brdc3540.14n -n 2913 -v -d 3600 -w -b 8
```

就能实现实时 GPS 发射了，但是目前只有 LimeSDR-USB 比较稳定，limesdr mini 实时发射不稳定，只能发射录制文件

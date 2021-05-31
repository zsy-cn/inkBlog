CLOCK_MONOTONIC 在 timerfd_create 以及 clock_gettime 中都有使用，具体函数如下：

int timerfd_create(int clockid, int flags);
//创建 timerfd 描述符
//clockid 可以填 CLOCK_REALTIME，CLOCK_MONOTONIC
//flags 可以填 0，O_CLOEXEC，O_NONBLOCK

函数： int clock_gettime(clockid_t clk_id, struct timespec \*tp);
//得到当前的时间
//clk_id 设置时间的类型
clockid_t： 用于指定计时时钟的类型，有以下几种类型：
CLOCK_REALTIME： 系统实时时间，从 Epoch 计时，可被设置更改。系统实时时间,随系统实时时间改变而改变,即从 UTC1970-1-1 0:0:0 开始计时,中间时刻如果系统时间被用户改成其他,则对应的时间相应改变
CLOCK_MONOTONIC： 系统运行时间，从系统启动时开始计时，不受系统时间被用户改变的影响,系统休眠时不再计时（NTP 与硬件时钟有问题时会影响其频率，没有验证过）。
CLOCK_PROCESS_CPUTIME_ID： 本进程启动到此刻使用 CPU 的时间，当使用 sleep 等函数时不再计时。
CLOCK_THREAD_CPUTIME_ID： 本线程启动到此刻使用 CPU 的时间，当使用 sleep 等函数时不再计时。
CLOCK_MONOTONIC_RAW ： 系统运行时间，从系统启动时开始计时，系统休眠时不再计时（NTP 与硬件时钟有问题时不会影响其频率，没有验证过）。
CLOCK_REALTIME_COARSE： 系统实时时间，从 Epoch 计时，可被设置更改，速度更快精度更低。
CLOCK_MONOTONIC_COARSE： 系统运行时间，从系统启动时开始计时，速度更快精度更低，系统休眠时不再计时（NTP 与硬件时钟有问题时会影响其频率，没有验证过）。
CLOCK_BOOTTIME： 与 CLOCK_MONOTONIC 类似
CLOCK_REALTIME_ALARM ： 闹钟时间（应该休眠后继续计时，没验证过），系统实时时间，从 Epoch 计时，可被设置更改。
CLOCK_BOOTTIME_ALARM： 闹钟时间（应该休眠后继续计时，没验证过），系统运行时间，从系统启动时开始计时。
CLOCK_TAI： 原子钟的时间，与 CLOCK_REALTIME 类似，不可被更改，没有闰秒。

```c
struct timespec：
struct timespec
{
time_t tv_sec; //秒
long tv_nsec; //纳秒
};
```

例子：

```c
#include <stdio.h>
#include <string.h>
#include <time.h>
#include <sys/time.h>
#include <assert.h>
#include <unistd.h>

static void clock_gettime_test()
{
struct timespec ts;

    clock_gettime(CLOCK_REALTIME, &ts);
    printf("CLOCK_REALTIME(1) sec = %lu, nsec = %ld\n", ts.tv_sec, ts.tv_nsec);
    usleep(1000 * 1000);
    clock_gettime(CLOCK_REALTIME, &ts);
    printf("CLOCK_REALTIME(2) sec = %lu, nsec = %ld\n", ts.tv_sec, ts.tv_nsec);

    clock_gettime(CLOCK_MONOTONIC, &ts);
    printf("CLOCK_MONOTONIC(1) sec = %lu, nsec = %ld\n", ts.tv_sec, ts.tv_nsec);
    usleep(1000 * 1000);
    clock_gettime(CLOCK_MONOTONIC, &ts);
    printf("CLOCK_MONOTONIC(2) sec = %lu, nsec = %ld\n", ts.tv_sec, ts.tv_nsec);

    clock_gettime(CLOCK_PROCESS_CPUTIME_ID, &ts);
    printf("CLOCK_PROCESS_CPUTIME_ID(1) sec = %lu, nsec = %ld\n", ts.tv_sec, ts.tv_nsec);
    usleep(1000 * 1000);
    clock_gettime(CLOCK_PROCESS_CPUTIME_ID, &ts);
    printf("CLOCK_PROCESS_CPUTIME_ID(2) sec = %lu, nsec = %ld\n", ts.tv_sec, ts.tv_nsec);

    clock_gettime(CLOCK_THREAD_CPUTIME_ID, &ts);
    printf("CLOCK_THREAD_CPUTIME_ID(1) sec = %lu, nsec = %ld\n", ts.tv_sec, ts.tv_nsec);
    usleep(1000 * 1000);
    clock_gettime(CLOCK_THREAD_CPUTIME_ID, &ts);
    printf("CLOCK_THREAD_CPUTIME_ID(2) sec = %lu, nsec = %ld\n", ts.tv_sec, ts.tv_nsec);

    clock_gettime(CLOCK_MONOTONIC_RAW, &ts);
    printf("CLOCK_MONOTONIC_RAW(1) sec = %lu, nsec = %ld\n", ts.tv_sec, ts.tv_nsec);
    usleep(1000 * 1000);
    clock_gettime(CLOCK_MONOTONIC_RAW, &ts);
    printf("CLOCK_MONOTONIC_RAW(2) sec = %lu, nsec = %ld\n", ts.tv_sec, ts.tv_nsec);

    clock_gettime(CLOCK_REALTIME_COARSE, &ts);
    printf("CLOCK_REALTIME_COARSE(1) sec = %lu, nsec = %ld\n", ts.tv_sec, ts.tv_nsec);
    usleep(1000 * 1000);
    clock_gettime(CLOCK_REALTIME_COARSE, &ts);
    printf("CLOCK_REALTIME_COARSE(2) sec = %lu, nsec = %ld\n", ts.tv_sec, ts.tv_nsec);

    clock_gettime(CLOCK_MONOTONIC_COARSE, &ts);
    printf("CLOCK_MONOTONIC_COARSE(1) sec = %lu, nsec = %ld\n", ts.tv_sec, ts.tv_nsec);
    usleep(1000 * 1000);
    clock_gettime(CLOCK_MONOTONIC_COARSE, &ts);
    printf("CLOCK_MONOTONIC_COARSE(2) sec = %lu, nsec = %ld\n", ts.tv_sec, ts.tv_nsec);

    clock_gettime(CLOCK_BOOTTIME, &ts);
    printf("CLOCK_BOOTTIME(1) sec = %lu, nsec = %ld\n", ts.tv_sec, ts.tv_nsec);
    usleep(1000 * 1000);
    clock_gettime(CLOCK_BOOTTIME, &ts);
    printf("CLOCK_BOOTTIME(2) sec = %lu, nsec = %ld\n", ts.tv_sec, ts.tv_nsec);

    clock_gettime(CLOCK_REALTIME_ALARM, &ts);
    printf("CLOCK_REALTIME_ALARM(1) sec = %lu, nsec = %ld\n", ts.tv_sec, ts.tv_nsec);
    usleep(1000 * 1000);
    clock_gettime(CLOCK_REALTIME_ALARM, &ts);
    printf("CLOCK_REALTIME_ALARM(2) sec = %lu, nsec = %ld\n", ts.tv_sec, ts.tv_nsec);

    clock_gettime(CLOCK_BOOTTIME_ALARM, &ts);
    printf("CLOCK_BOOTTIME_ALARM(1) sec = %lu, nsec = %ld\n", ts.tv_sec, ts.tv_nsec);
    usleep(1000 * 1000);
    clock_gettime(CLOCK_BOOTTIME_ALARM, &ts);
    printf("CLOCK_BOOTTIME_ALARM(2) sec = %lu, nsec = %ld\n", ts.tv_sec, ts.tv_nsec);

    clock_gettime(CLOCK_TAI, &ts);
    printf("CLOCK_TAI(1) sec = %lu, nsec = %ld\n", ts.tv_sec, ts.tv_nsec);
    usleep(1000 * 1000);
    clock_gettime(CLOCK_TAI, &ts);
    printf("CLOCK_TAI(2) sec = %lu, nsec = %ld\n", ts.tv_sec, ts.tv_nsec);

}

int main()
{
clock_gettime_test();

    return 0;

}
```

```c
#include <stdio.h>
#include <dirent.h>
#include <iostream>
#include <sys/time.h>
#include <sys/types.h>
int kMicroSecondsPerSecond = 1000 _ 1000;
int64_t NMicroSecondsPerSecond = 1000 _ 1000 _ 1000;
int64_t now1()
{
struct timeval tv;
gettimeofday(&tv, NULL);//gettimeofday 就是 clock_gettime（CLOCK_MONOTONIC）的用户层封
//装，并且降低了 clock_gettime 函数的精度，clock_gettime 能精确到纳秒级，而 gettimeofday 精确到微妙级
int64_t seconds = tv.tv_sec;
printf("%ld\n",seconds);
return seconds _ kMicroSecondsPerSecond + tv.tv_usec;
//返回一个 Timestamp 结构体，相当于创建一个当前时间的 Timestamp 结构体
}

int64_t now2()
{
struct timespec tv;
clock_gettime(CLOCK_MONOTONIC, &tv);//gettimeofday 就是 clock_gettime（CLOCK_MONOTONIC）的用户层封
//装，并且降低了 clock_gettime 函数的精度，clock_gettime 能精确到纳秒级，而 gettimeofday 精确到微妙级
int64_t seconds = tv.tv_sec;
printf("%ld\n",seconds);
return seconds \* kMicroSecondsPerSecond + tv.tv_nsec/1000;
//返回一个 Timestamp 结构体，相当于创建一个当前时间的 Timestamp 结构体
}

void translate(int64*t microSecondsSinceEpoch*)
{
char buf[32] = {0};
time*t seconds = static_cast<time_t>(microSecondsSinceEpoch* / kMicroSecondsPerSecond);
int microseconds = static*cast<int>(microSecondsSinceEpoch* % kMicroSecondsPerSecond);
struct tm tm_time;
gmtime_r(&seconds, &tm_time);//将总秒数转换成————年-月-日-小时-分-秒为单位，并且还会自动加上 1970 年 1 月 1 日时间
snprintf(buf, sizeof(buf), "%4d%02d%02d %02d:%02d:%02d.%06d",
tm_time.tm_year+1900, tm_time.tm_mon + 1, tm_time.tm_mday,
tm_time.tm_hour, tm_time.tm_min, tm_time.tm_sec,
microseconds);//总秒数加上 1900 年 1 月 1 日然后转换成固定格式
printf("%s\n",buf);
}

int main()
{
int64*t microSecondsSinceEpoch1* = now1();
translate(microSecondsSinceEpoch1*);
int64_t microSecondsSinceEpoch2* = now2();
translate(microSecondsSinceEpoch2\_);
translate(0);
return 0;
}
```

```输出
1574434622
20191122 14:57:02.710567
52874
19700101 14:41:14.286180
19700101 00:00:00.000000
```

所以 clock_gettime(CLOCK_MONOTONIC, &tv); 就是开机到现在的时间，而 gettimeofday 则是从 1970 年开始，到现在的时间，但是这个时间是零时区的事件，也就是评论所说的格林尼治时间。

https://www.runoob.com/w3cnote/cpp-time_t.html 可以参考这个网页

注意：其实在 timerfd 这套函数中，CLOCK_MONOTONIC 具体代表什么含义并不重要，因为在设置 timerfd 的到期时间时，使用的函数如下：

int timerfd_settime(int fd,
int flags,
const struct itimerspec *new_value,
struct itimerspec *old_value);

其中 flags 填写 1（TFD_TIMER_ABSTIME）代表绝对时间，0 代表相对时间

如果 timerfd_settime 第二个参数设置为 0,new_value.it_value 设置为 1
如果 timerfd_settime 第二个参数设置为 TFD_TIMER_ABSTIME,new_value.it_value 设置为 now.tv_sec + 1。

并且 timerfd_settime 的当前时间可以使用函数 clock_gettime 来获取，填写的参数要和创建 timerfd 时的参数一样，也就是 clk_id 和 clockid 一样

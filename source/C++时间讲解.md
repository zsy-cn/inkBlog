c++ 时间类型详解 time_t
分类 编程技术

Unix 时间戳(Unix timestamp)，或称 Unix 时间(Unix time)、POSIX 时间(POSIX time)，是一种时间表示方式，定义为从格林威治时间 1970 年 01 月 01 日 00 时 00 分 00 秒起至现在的总秒数。Unix 时间戳不仅被使用在 Unix 系统、类 Unix 系统中，也在许多其他操作系统中被广告采用。

目前相当一部分操作系统使用 32 位二进制数字表示时间。此类系统的 Unix 时间戳最多可以使用到格林威治时间 2038 年 01 月 19 日 03 时 14 分 07 秒（二进制：01111111 11111111 11111111 11111111）。其后一秒，二进制数字会变为 10000000 00000000 00000000 00000000，发生溢出错误，造成系统将时间误解为 1901 年 12 月 13 日 20 时 45 分 52 秒。这很可能会引起软件故障，甚至是系统瘫痪。使用 64 位二进制数字表示时间的系统（最多可以使用到格林威治时间 292,277,026,596 年 12 月 04 日 15 时 30 分 08 秒）则基本不会遇到这类溢出问题。

首先我们了解一下时间的相关概念，以及之间的区别，需要了解的时间概念有：

    本地时间(locale time)
    格林威治时间（Greenwich Mean Time GMT）
    时间协调时间 （Universal Time Coordinated UTC）

本地时间，显而易见不用解释了

先看看时间的标准：

（1）世界时

世界时是最早的时间标准。在 1884 年，国际上将 1s 确定为全年内每日平均长度的 1/8.64×104。以此标准形成的时间系统，称为世界时，即 UT1。1972 年国际上开始使用国际原子时标，从那以后，经过格林威治老天文台本初子午线的时间便被称为世界时，即 UT2，或称格林威治时间（GMT），是对地球转速周期性差异进行校正后的世界时。

（2）原子时

1967 年，人们利用铯原子振荡周期极为规律的特性，研制出了高精度的原子时钟，将铯原子能级跃迁辐射 9192631770 周所经历的时间定为 1s。现在用的时间就是 1971 年 10 月定义的国际原子时，是通过世界上大约 200 多台原子钟进行对比后，再由国际度量衡局时间所进行数据处理，得出的统一的原子时，简称 TAI。

（3）世界协调时

世界协调时是以地球自转为基础的时间标准。由于地球自转速度并不均匀，并非每天都是精确的 86400 原子 s，因而导致了自转时间与世界时之间存在 18 个月有 1s 的误差。为纠正这种误差，国际地球自转研究所根据地球自转的实际情况对格林威治时间进行增减闰 s 的调整，与国际度量衡局时间所联合向全世界发布标准时间，这就是所谓的世界协调时（UTC:CoordinatdeUniversalTime）。UTC 的表示方式为：年（y）、月（m）、日（d）、时（h）、分（min）、秒（s），均用数字表示。

GPS 系统中有两种时间区分，一为 UTC，另一为 LT（地方时）两者的区别为时区不同，UTC 就是 0 时区的时间，地方时为本地时间，如北京为早上八点（东八区），UTC 时间就为零点，时间比北京时晚八小时，以此计算即可通过上面的了解，我们可以认为格林威治时间就是时间协调时间（GMT=UTC），格林威治时间和 UTC 时间均用秒数来计算的。

而在我们平时工作当中看到的计算机日志里面写的时间大多数是用 UTC 时间来计算的，那么我们该怎么将 UTC 时间转化为本地时间便于查看日志，那么在作程序开发时又该怎么将本地时间转化为 UTC 时间呢？

下面就介绍一个简单而使用的工具，就是使用 linux/unix 命令 date 来进行本地时间和 local 时间的转化。

大家都知道，在计算机中看到的 utc 时间都是从（1970 年 01 月 01 日 0:00:00)开始计算秒数的。所看到的 UTC 时间那就是从 1970 年这个时间点起到具体时间共有多少秒。

我们在编程中可能会经常用到时间，比如取得系统的时间（获取系统的年、月、日、时、分、秒，星期等），或者是隔一段时间去做某事，那么我们就用到一些时间函数。

linux 下存储时间常见的有两种存储方式，一个是从 1970 年到现在经过了多少秒，一个是用一个结构来分别存储年月日时分秒的。

time_t 这种类型就是用来存储从 1970 年到现在经过了多少秒，要想更精确一点，可以用结构 struct timeval，它精确到微妙。

struct timeval
{
long tv*sec; /*秒*/
long tv_usec; /*微秒\_/
};

而直接存储年月日的是一个结构：

struct tm
{
int tm*sec; /*秒，正常范围 0-59， 但允许至 61*/
int tm_min; /*分钟，0-59*/
int tm_hour; /*小时， 0-23*/
int tm_mday; /*日，即一个月中的第几天，1-31*/
int tm_mon; /*月， 从一月算起，0-11*/ 1+p->tm_mon;
int tm_year; /*年， 从 1900 至今已经多少年*/ 1900 ＋ p->tm_year;
int tm_wday; /*星期，一周中的第几天， 从星期日算起，0-6*/
int tm_yday; /*从今年 1 月 1 日到目前的天数，范围 0-365*/
int tm_isdst; /*日光节约时间的旗标\_/
};

需要特别注意的是，年份是从 1900 年起至今多少年，而不是直接存储如 2011 年，月份从 0 开始的，0 表示一月，星期也是从 0 开始的， 0 表示星期日，1 表示星期一。

下面介绍一下我们常用的时间函数：

#include <time.h>
char _asctime(const struct tm_ timeptr);

将结构中的信息转换为真实世界的时间，以字符串的形式显示

char *ctime(const time_t *timep);

将 timep 转换为真是世界的时间，以字符串显示，它和 asctime 不同就在于传入的参数形式不一样

double difftime(time_t time1, time_t time2);

返回两个时间相差的秒数

int gettimeofday(struct timeval *tv, struct timezone *tz);

返回当前距离 1970 年的秒数和微妙数，后面的 tz 是时区，一般不用

struct tm* gmtime(const time_t *timep);

将 time_t 表示的时间转换为没有经过时区转换的 UTC 时间，是一个 struct tm 结构指针

stuct tm* localtime(const time_t *timep);

和 gmtime 类似，但是它是经过时区转换的时间。

time_t mktime(struct tm\* timeptr);

将 struct tm 结构的时间转换为从 1970 年至今的秒数

time_t time(time_t \*t);

取得从 1970 年 1 月 1 日至今的秒数。

上面是简单的介绍，下面通过实战来看看这些函数的用法：

/_gettime1.c_/
#include <time.h>

int main()
{
time_t timep;

    time(&timep); /*获取time_t类型的当前时间*/
    /*用gmtime将time_t类型的时间转换为struct tm类型的时间按，／／没有经过时区转换的UTC时间
      然后再用asctime转换为我们常见的格式 Fri Jan 11 17:25:24 2008
    */
    printf("%s", asctime(gmtime(&timep)));
    return 0;

}

编译并运行：

$gcc -o gettime1 gettime1.c
$./gettime1
Fri Jan 11 17:04:08 2008

下面是直接把 time_t 类型的转换为我们常见的格式:

/_ gettime2.c_/
#include <time.h>

int main()
{
time_t timep;

    time(&timep); /*获取time_t类型当前时间*/
    /*转换为常见的字符串：Fri Jan 11 17:04:08 2008*/
    printf("%s", ctime(&timep));
    return 0;

}

编译并运行：

$gcc -o gettime2 gettime2.c
$./gettime2
Sat Jan 12 01:25:29 2008

我看了一本书上面说的这两个例子如果先后执行的话，两个的结果除了秒上有差别之外（执行程序需要时间），应该是一样的，可是我这里执行却发现差了很长时间按，一个是周五，一个是周六，后来我用 date 命令执行了一遍

$ date
六 1 月 12 01:25:19 CST 2008

我发现 date 和 gettime2 比较一致， 我估计可能 gettime1 并没有经过时区的转换，它们是有差别的。

/_gettime3.c _/
#include <time.h>

int main()
{
char *wday[] = {"Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"};
time_t timep;
struct tm *p;

    time(&timep); /*获得time_t结构的时间，UTC时间*/
    p = gmtime(&timep); /*转换为struct tm结构的UTC时间*/
    printf("%d/%d/%d ", 1900 + p->tm_year, 1+ p->tm_mon, p->tm_mday);
    printf("%s %d:%d:%d\n", wday[p->tm_wday], p->tm_hour,
        p->tm_min, p->tm_sec);
    return 0;

}

编译并运行：

$gcc -o gettime3 gettime3.c
$./gettime3
2008/1/11 Fri 17:42:54

从这个时间结果上来看，它和 gettime1 保持一致。

/_gettime4.c_/
#include <time.h>

int main()
{
char *wday[] = {"Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"};
time_t timep;
struct tm *p;

    time(&timep); /*获得time_t结构的时间，UTC时间*/
    p = localtime(&timep); /*转换为struct tm结构的当地时间*/
    printf("%d/%d/%d ", 1900 + p->tm_year, 1+ p->tm_mon, p->tm_mday);
    printf("%s %d:%d:%d\n", wday[p->tm_wday], p->tm_hour, p->tm_min, p->tm_sec);
    return 0;

}

编译并运行：

$gcc -o gettime4 gettime4.c
$./gettime4
2008/1/12 Sat 1:49:29

从上面的结果我们可以这样说：

time, gmtime, asctime 所表示的时间都是 UTC 时间，只是数据类型不一样，

而 localtime, ctime 所表示的时间都是经过时区转换后的时间，它和你用系统命令 date 所表示的 CST 时间应该保持一致。

/_gettime5.c_/
#include <time.h>

int main()
{
time_t timep;
struct tm \*p;

    time(&timep); /*当前time_t类型UTC时间*/
    printf("time():%d\n",timep);
    p = localtime(&timep); /*转换为本地的tm结构的时间按*/
    timep = mktime(p); /*重新转换为time_t类型的UTC时间，这里有一个时区的转换*/ //by lizp 错误，没有时区转换， 将struct tm 结构的时间转换为从1970年至p的秒数
    printf("time()->localtime()->mktime(): %d\n", timep);
    return 0;

}

编译并运行：

$gcc -o gettime5 gettime5.c
$./gettime5
time():1200074913
time()->localtime()->mktime(): 1200074913

这里面把 UTC 时间按转换为本地时间，然后再把本地时间转换为 UTC 时间，它们转换的结果保持一致。

/_gettime6.c _/
#include <time.h>

int main()
{
time_t timep;
struct tm \*p;

    time(&timep);  /*得到time_t类型的UTC时间*/
    printf("time():%d\n",timep);
    p = gmtime(&timep); /*得到tm结构的UTC时间*/
    timep = mktime(p); /*转换，这里会有时区的转换*/ //by lizp 错误，没有时区转换， 将struct tm 结构的时间转换为从1970年至p的秒数
    printf("time()->gmtime()->mktime(): %d\n", timep);
    return 0;

}

编译并运行：

$gcc -o gettime6 gettime6.c
$./gettime6
time():1200075192
time()->gmtime()->mktime(): 1200046392

从这里面我们可以看出，转换后时间不一致了，计算一下，整整差了 8 个小时( (1200075192-1200046392)/3600 = 8)，说明 mktime 会把本地时间转换为 UTC 时间，这里面本来就是 UTC 时间，于是再弄个时区转换，结果差了 8 个小时，用的时候应该注意。

strftime() 函数将时间格式化

我们可以使用 strftime（）函数将时间格式化为我们想要的格式。它的原型如下：

size_t strftime(
char *strDest,
size_t maxsize,
const char *format,
const struct tm \*timeptr
);

我们可以根据 format 指向字符串中格式命令把 timeptr 中保存的时间信息放在 strDest 指向的字符串中，最多向 strDest 中存放 maxsize 个字符。该函数返回向 strDest 指向的字符串中放置的字符数。

函数 strftime()的操作有些类似于 sprintf()：识别以百分号(%)开始的格式命令集合，格式化输出结果放在一个字符串中。格式化命令说明串 strDest 中各种日期和时间信息的确切表示方法。格式串中的其他字符原样放进串中。格式命令列在下面，它们是区分大小写的。

%a 星期几的简写
%A 星期几的全称
%b 月分的简写
%B 月份的全称
%c 标准的日期的时间串
%C 年份的后两位数字
%d 十进制表示的每月的第几天
%D 月/天/年
%e 在两字符域中，十进制表示的每月的第几天
%F 年-月-日
%g 年份的后两位数字，使用基于周的年
%G 年分，使用基于周的年
%h 简写的月份名
%H 24 小时制的小时
%I 12 小时制的小时
%j 十进制表示的每年的第几天
%m 十进制表示的月份
%M 十时制表示的分钟数
%n 新行符
%p 本地的 AM 或 PM 的等价显示
%r 12 小时的时间
%R 显示小时和分钟：hh:mm
%S 十进制的秒数
%t 水平制表符
%T 显示时分秒：hh:mm:ss
%u 每周的第几天，星期一为第一天 （值从 0 到 6，星期一为 0）
%U 第年的第几周，把星期日做为第一天（值从 0 到 53）
%V 每年的第几周，使用基于周的年
%w 十进制表示的星期几（值从 0 到 6，星期天为 0）
%W 每年的第几周，把星期一做为第一天（值从 0 到 53）
%x 标准的日期串
%X 标准的时间串
%y 不带世纪的十进制年份（值从 0 到 99）
%Y 带世纪部分的十制年份
%z，%Z 时区名称，如果不能得到时区名称则返回空字符。
%% 百分号

如果想显示现在是几点了，并以 12 小时制显示，就象下面这段程序：

#include "time.h"
#include "stdio.h"
int main(void)
{
struct tm \*ptr;
time_t lt;
char str[80];
lt=time(NULL);
ptr=localtime(<);
strftime(str,100,"It is now %I %p",ptr);
printf(str);
return 0;
}

其运行结果为：

It is now 4PM

而下面的程序则显示当前的完整日期：

#include<stdio.h>
#include<string.h>
#include<time.h>
int main( void )
{
struct tm \*newtime;
char tmpbuf[128];
time_t lt1;

    time( &lt1 );
    newtime=localtime(&lt1);

    strftime( tmpbuf, 128, "Today is %A, day %d of %B in the year %Y.\n", newtime);
    printf(tmpbuf);

    return 0;

}

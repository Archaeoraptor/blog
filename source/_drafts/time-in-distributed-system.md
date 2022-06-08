---
title: Lamport时间戳：分布式系统中的时间
date: 2022-03-08 15:27:36
tags:
- timestamp
- Lamport
abbrlink: 'time-in-distributed-system'
categories:
- Concurrency&Go
---
世界始于1970.1.1
<!-- more -->

## Lamport时间戳

如果看过Paxos或者Raft, 是不是有种似曾相识的感觉？在没有一个统一的可靠时间

## 时间

### 晶振时间的产生和校准

曾经家里有个挂在墙上的钟，每天都要快几秒。每过几个月都要调一下。后来家里看时间逐渐变成了看手机，我以为再也不用苦哈哈的对时间了。后来遇到了STM32的RTC日历，真的谁做谁麻，晶振如果频率不太行一天能差个几分钟，严重的一小时差一分钟都不奇怪。

（跑个题，人的内源性节律一般比24小时长一点。如果你靠光线和闹钟之类的东西调节你的节律，你可能会睡得越来越晚）

好在分布式系统的服务器一般不像嵌入式的板子那样为了节省成本用那种廉价晶振，现在使用NTP校时和物理时钟在大多数服务器上已经足够将时间差缩小到50ms以内，如果不是对延迟有严格要求，那直接所有节点用一个全局的时钟就好了。

一般主板里面会有纽扣电池供电维护RTC时间（Real Time Clock），不过这个RTC不是很准（虽然大部分电脑和服务器的晶振比stm32内置的那种廉价低频晶振好多了），一般只在待机的时候使用，开机之后换成PIT时间。（stm32开机后用systick或者TIM，又跑题了，下面不说stm32了）

PIT（Programmable Interval Timer），可编程间隔定时器，也是一个晶振芯片啦（比如intel8253）。用晶振产生一个固定频率的高频脉冲，然后进行分频。分频后输出的信号的时钟周期成为一个时钟滴答（clock cycle）。

Linux用全局变量`jiffies`维护从开机到现在的时钟滴答数目。linux默认时钟频率（Timer Frequency）是250hz，部分桌面发行版为了实时响应会设置成1000Hz。提高时钟频率处理中断更快了（如果100Hz）

```bash
$zcat /proc/config.gz | grep CONFIG_HZ
CONFIG_HZ_1000=y
CONFIG_HZ=1000
```

### 时间的格式和约定

时间开始于1970年1月1日，Unix时间戳和UTC都是从1970开始算的。

```bash
$ date -u +"%Y-%m-%dT%H:%M:%SZ"
2022-03-08T09:15:06Z
$ date +%s
1646731009
```

现在我除非要打印出来给人看，一律Unix时间戳，省了很多日期格式换算的麻烦。

### linux中的time系统调用

需要注意localtime不是线程安全的，要用localtime_r

### 关于go的time包

go time 包的标准时间格式挺迷的，不是yymmdd那种，是"01/02 03:04:05PM '06 -0700"，意味着"2006年01月02日15时04分05秒"。后来查了一下这个时间是Unix时间戳``，用`date`命令打出来是

```bash
$TZ=GMT+7:00 date -d @1136239445 # 这里是GMT+7.00不是GMT-7.00
Mon Jan 2 15:04:05 MST 2006
```

星期一二月下午三点4分5秒06年-7时区，行吧，你说是那就是吧。

go官方的time包，还要小心时区问题：`time.Now()`返回的是本地时间，`time.Parse`返回的是UTC +0 时间，如果要parse出本地时间，那需要用`ParseInLocation`。

```go
loc, _ := time.LoadLocation("Asia/Shanghai")
t, _ = time.ParseInLocation(shortForm, "2012-Jul-09", loc)
```

咱也不知道为啥不把时间的`parse`默认设成本地的，当年用docker的时候就被时区坑了一次。绝大多数人用到时间转换应该的都是本地时间吧？？？

设置一个发送心跳用的定时器直接用`time.Tick(time.Second)`

### 直接用NTP等，认为全局时间一致

Coackroach就是直接500ms（或者）的时间偏移，然后

## 链接

[高精度事件计时器](https://zh.wikipedia.org/wiki/高精度事件计时器)  
[How fast should HZ be?](https://lwn.net/Articles/145973/)   
[分布式事务中的时间戳](https://ericfu.me/timestamp-in-distributed-trans/)  

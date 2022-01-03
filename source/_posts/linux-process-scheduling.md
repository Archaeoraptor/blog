---
title: Linux进程调度，从服务器到桌面
date: 2021-08-30 11:26:55
tags:
- Linux
- scheduler
abbrlink: 'linux-processing-scheduling'
categories:
 - Linux&Unix
---
用了一周linux-ck和MuQSS，发现没有想象中的那么好。哦，这还要从一个麻醉师说起。

<!-- more -->

没仔细看调度器之前我以为Linux进程调度都是和内存管理一样很复杂的东西。实现比较简单的进程调度就几十行代码。CFS这种比较复杂的就不仔细说了、多核和分布式调度这些比较复杂的下面也不会多讲。（主要是我水平太菜了，怕讲错）

## 进程调度简介

进程是操作系统虚拟化出来的东西，

Linux里面基本不区分进程和线程，统称为task（任务）。下面的进程等词都指task。

CPU的资源是有限的，进程之间需要一个调度器来分配资源使用。最常见的调度器是Linux内核主线默认采用的CFS。

注：不要只盯着复杂度O(n), O(1), O(log n), 效果还是得看实测。引用一下cauche调度器readme里的一段话

O(n) sounds scary, but usually for a machine with 4 CPUS where it is used for desktop or mobile jobs, the maximum number of runnable tasks might not exceeds 10 (at the pick next run time) - the idle tasks are excluded since they are dequeued when sleeping and enqueued when they wake up.

在电脑桌面和手机、4核CPU使用场景，最多运行任务不超过10个。所以复杂度O(n)不一定比O(1)调度速度快。

### 关于进程

在`include/linux/sched.h`头文件里面有进程的定义（一个结构体来表示的task）

```c
struct task_struct
```

Linux系统在启动的时候会首先执行`start_kernel()`函数[init/main.c](https://elixir.bootlin.com/linux/v5.7-rc1/source/init/main.c)。

首先是`set_task_stack_end_magic(&init_task)`创建一个叫`init_task`的进程，一般管它叫0号进程（也就是后面的idle进程）。

然后调用`sched_init();`函数初始化进程调度。

然后在`start_kernel()`函数最后调用`rest_init`, 两个`kernel_thread`分别创建了一号进程（`init`）和二号进程(`kthreadd`)。

```c
noinline void __ref rest_init(void)
{
	struct task_struct *tsk;
	int pid;

	rcu_scheduler_starting();
    ...........
	pid = kernel_thread(kernel_init, NULL, CLONE_FS);
	...........
	pid = kernel_thread(kthreadd, NULL, CLONE_FS | CLONE_FILES);
	...........
}
```

一般1号进程是用户态进程，整个用户态的进程树都是它`fork`出来的。关于1号进程的介绍可以看看systemd作者写的[[Rethinking PID 1](http://0pointer.net/blog/projects/systemd.html)](http://0pointer.de/blog/projects/systemd.html) 中文翻译: [重新思考 1 号进程](http://kaiwangchen.github.io/2014/10/08/systemd.html)

2号进程是内核进程，负责管理和调度内核线程。

进程描述符里面有一些调度需要的信息，比如。

### 初代调度器

初代调度器非常简单，就是一个runqueue（运行队列），源码在[kernel/sched.c](https://elixir.bootlin.com/linux/0.11/source/kernel/sched.c),   我们忽略掉系统调用和printk日志以及floppy之类的，只看调度部分：

```c
/*
 *  'schedule()' is the scheduler function. This is GOOD CODE! There
 * probably won't be any reason to change this, as it should work well
 * in all circumstances (ie gives IO-bound processes good response etc).
 * The one thing you might take a look at is the signal-handler code here.
 *
 *   NOTE!!  Task 0 is the 'idle' task, which gets called when no other
 * tasks can run. It can not be killed, and it cannot sleep. The 'state'
 * information in task[0] is never used.
 */
void schedule(void)
{
	int i,next,c;
	struct task_struct ** p;

/* check alarm, wake up any interruptible tasks that have got a signal */

	for(p = &LAST_TASK ; p > &FIRST_TASK ; --p)
		if (*p) {
			if ((*p)->alarm && (*p)->alarm < jiffies) {
					(*p)->signal |= (1<<(SIGALRM-1));
					(*p)->alarm = 0;
				}
			if (((*p)->signal & ~(_BLOCKABLE & (*p)->blocked)) &&
			(*p)->state==TASK_INTERRUPTIBLE)
				(*p)->state=TASK_RUNNING;
		}

/* this is the scheduler proper: */

	while (1) {
		c = -1;
		next = 0;
		i = NR_TASKS;
		p = &task[NR_TASKS];
		while (--i) {
			if (!*--p)
				continue;
			if ((*p)->state == TASK_RUNNING && (*p)->counter > c)
				c = (*p)->counter, next = i;
		}
		if (c) break;
		for(p = &LAST_TASK ; p > &FIRST_TASK ; --p)
			if (*p)
				(*p)->counter = ((*p)->counter >> 1) +
						(*p)->priority;
	}
	switch_to(next);
}
```

就只有这么几行。也没有nice值和多核，甚至没有生命周期，还是很容易理解的。

jiffies是系统开机以来tick的次数（alarm>jiffies说明过期了，重置为0）

counter是时间片，单位是tick（时钟滴答），调度器根据couter大小决定优先级（couter越大优先级越高）

NR_TASKS是task（进程）总数。

第一个循环是检查一遍[alarm()](https://man7.org/linux/man-pages/man2/alarm.2.html)函数，唤醒任何收到alarm传来的signal的没有被阻塞的tasks，将`TASK_INTERRUPTIBLE`（挂起）改为`TASK_RUNNING`可执行。

然后`while(1)` 这个死循环一直执行到关机，每次循环先`while (--i)`找出counter（时间片）最大的task。（couter越大说明）

`if (c) break;`和下面的这些是说如果c为0（所有进程的counter用完了），就重新分配counter。

最后调用`switch_to(next)`切换进程。（切换到counter最大的一个）

然后还有几个函数，是几个状态的转换，也很简单。

```c
int sys_pause(void)
{
	current->state = TASK_INTERRUPTIBLE;
	schedule();
	return 0;
}

void sleep_on(struct task_struct **p)
{
	struct task_struct *tmp;

	if (!p)
		return;
	if (current == &(init_task.task))
		panic("task[0] trying to sleep");
	tmp = *p;
	*p = current;
	current->state = TASK_UNINTERRUPTIBLE;
	schedule();
	if (tmp)小
		tmp->state=0;
}

void interruptible_sleep_on(struct task_struct **p)
{
	struct task_struct *tmp;

	if (!p)
		return;
	if (current == &(init_task.task))
		panic("task[0] trying to sleep");
	tmp=*p;
	*p=current;
repeat:	current->state = TASK_INTERRUPTIBLE;
	schedule();
	if (*p && *p != current) {
		(**p).state=0;
		goto repeat;
	}
	*p=NULL;
	if (tmp)
		tmp->state=0;
}

void wake_up(struct task_struct **p)
{
	if (p && *p) {
		(**p).state=0;
		*p=NULL;
	}
}
```

这个调度器复杂度是O(n)的（复杂度没啥用，因为NR_TASKS早期特别小）。只有分配的counter值作为调度优先级

0号进程的优先级是最低的（最后被调度）。在后面的调度器中0号进程根本不参与调度。

### O(n)调度器

早期的调度器是和UNIX的差不多，就是O(n)调度器。

O(n)调度器是用runqueue（运行队列），和初代调度器不同的是CPU每个核都有一个runqueue。

### O(1)调度器

2.6版本

O(1)调度器比较适合服务器，基本上做到了将I/O利用率最大化。

静态优先级（一般叫nice值）

### CFS

CFS调度器（Completely Fair Scheduler），用的最广的一个（从2.6开始Linux内核主线默认就是他）。CFS的意思是完全公平调度器，完全公平是说每一个进程在一个周期时间内运行相同的时间。在一个生命周期`T`内，`N`个task占用CPU的时间均为$T/N$

CFS是一个红黑树实现的。是几种调度器中实现很复杂的一个调度器，源码在这里：[sched_fair.c Linux Kernel Source](https://elixir.bootlin.com/linux/latest/source/kernel/sched/fair.c)

文档在这里：https://www.kernel.org/doc/Documentation/scheduler/sched-design-CFS.txt

调度实体sched_entity，虚拟运行时间vruntime



### RT调度器

实时调度器，Real Time Scheduler。一般使用优先级队列（priority queue）实现的

进程根据优先级（priority）

实时调度器调度的实时进程的优先级通常很高（优先级0-99，不同内核可能不太一样）

非实时的进程优先级在100-139

执行`ps`命令可以查看优先级，PRI这一列是优先级（priority），NI这一列是NICE值

```bash
$ps -el
F S   UID     PID    PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
4 R     0       1       0  0  80   0 - 25249 -      ?        00:00:07 systemd
1 S     0       2       0  0  80   0 -     0 -      ?        00:00:00 kthreadd
1 I     0       3       2  0  60 -20 -     0 -      ?        00:00:00 rcu_gp
1 I     0       4       2  0  60 -20 -     0 -      ?        00:00:00 rcu_par_gp
1 I     0       6       2  0  60 -20 -     0 -      ?        00:00:00 kworker/0:0H-events_highpri
1 I     0       8       2  0  60 -20 -     0 -      ?        00:00:00 mm_percpu_wq
1 S     0       9       2  0  80   0 -     0 -      ?        00:00:00 rcu_tasks_rude_
1 S     0      10       2  0  80   0 -     0 -      ?        00:00:00 rcu_tasks_trace
1 S     0      11       2  0  80   0 -     0 -      ?        00:00:08 ksoftirqd/0
1 I     0      12       2  0  80   0 -     0 -      ?        00:00:18 rcu_sched
1 S     0      13       2  0 -40   - -     0 -      ?        00:00:00 migration/0
1 S     0      14       2  0   9   - -     0 -      ?        00:00:00 idle_inject/0
1 S     0      16       2  0  80   0 -     0 -      ?        00:00:00 cpuhp/0
5 S     0      17       2  0  80   0 -     0 -      ?        00:00:00 cpuhp/1
1 S     0      18       2  0   9   - -     0 -      ?        00:00:00 idle_inject/1
```

### 多核

CPU的每个核都有一个runqueue

### 组调度

本来打算先跳过这一部分的，以后写cgroup的时候再说。但是这个东西对桌面的优化很重要，比如这个很老的补丁。

[The ~200 Line Linux Kernel Patch That Does Wonders](https://www.phoronix.com/scan.php?page=article&item=linux_2637_video&num=1)

[Re: [RFC/RFT PATCH v3] sched: automated per tty task groups](https://marc.info/?l=linux-kernel&m=128978361700898&w=2) 

现在Linux的调度策略比较多，不同用户通过组调度进行资源的分配和隔离。说到这里你是不是想起了cgroup，对，这个东西就是用cgroup实现的。

### 其他的调度器

官方不支持调度器像kernel module一样做成可插拔的，所以其他魔改的调度策略通常单独做一个内核。

## 适合桌面的进程调度

Linux的一些设计和参数偏向服务器、嵌入式设备等用途，毕竟普通桌面用户的意见加起来都比不上半个红帽。
IO调度没关系，反正有个SSD就完全够了，用NOOP调度甚至直接不要IO调度都没关系。
进程调度就不太行了，默认的CFS (Completely Fair Scheduler)调度非常不适合日常桌面使用。
通常服务器多是I/O密集型任务，而桌面（Desktop Enviroment，比如KDE和GNOME这种）需要快速响应（低延迟）和不断切换任务，CFS就不适合了（没法做到不断的切换优先级和抢占）。

比如编译、看视频CPU硬解码、玩游戏，负载高了之后，如果用CFS调度桌面会非常卡，极大影响体验。

好吧这不太符合Unix哲学，你在terminal里面用管道

麻醉师Con Kolivas针对桌面系统做的BFS（后来改名叫MuQSS了）

https://www.linux-magazine.com/var/linux_magazin/storage/images/media/linux-magazine-eng-us/images/news-images/cartoon-features/367908-1-eng-US/Cartoon-Features_medium.png

## 服务器进程调度

大多数服务器默认的CFS以及足够合适了。

https://github.com/Tencent/TencentOS-kernel#离线调度算法bt

## 更改进程调度

### chrt更改调度策略

使用见[使用chrt修改进程调度策略及优先级](http://blog.lujun9972.win/blog/2018/08/28/使用chrt修改进程调度策略及优先级/index.html)

### 内核选择

*选择非官方支持的内核请谨慎，因为可能有很多不兼容的问题。*没有特别需求和强迫症还是推荐使用有官方支持的内核，比如Archlinux官方源支持维护``linux`（没啥特殊偏好就这个呗），`linux-lts`（lts版，不想天天更新内核就用这个）， `linux-zen`（跟AMD的zen关系不太大，主要是针对桌面的），`linux-harended`（有安全加固，会牺牲一定性能）

`linux`这个包的调度器是CFS，大多数情况下表现都不差。（lts当然也是喽）

如果你是桌面用户，对性能和延迟有需求，可以试试linux-ck、linux-zen、linux-xanmod、linux-tgk等一堆针对桌面和性能优化的内核。`linux-ck`的调度器是MuQSS。xanmod的调度器可以选cauche。

我个人体验是xanmod内核的延迟最低（直接拯救了我玩空洞骑士时肉眼可见的延迟）。玩OSU音游的喜欢用linux-zen，这个包是官方维护，比较省心，xanmod延迟虽然跑分上比这个低，对延迟要求不苛刻、反应慢的，可能体验不出来太大区别。

（音游在Linux上的声音延迟应该是PulseAudio的锅，但是现在Pipewire又不稳，实际效果还不如PulseAudio,等pipewire稳定下来不知道要多久，只有打点补丁才能维持的了生活的样子。感谢这位玩家给wine-osu打的补丁[Low-latency osu! on Linux](https://blog.thepoon.fr/osuLinuxAudioLatency/)）  
*关于游戏的题外话*
Archlinux可以装gamemode和performance-tweaks（这个在chaotic-aur里面）。

xanmod实测见：[XanMod's Linux 5.10 Kernel Helping Tap Extra Performance With The AMD Ryzen 9 5900X](https://www.phoronix.com/scan.php?page=article&item=xanmod-liquorix-510)
cuache相关讨论：https://forum.endeavouros.com/t/introducing-the-cacule-scheduler-a-cfs-replacement/13644

如果要用btrfs或者cgroups，请注意MuQSS兼容性不太好。

Archlinux直接AUR编译就行了。不过很多人换ck内核是因为老机器性能不好，找一个编译好的源（比如repo-ck）或者在别的机子上编译一个吧。

## 链接

https://lwn.net/Articles/720227/
https://www.linux-magazine.com/Online/News/Con-Kolivas-Introduces-New-BFS-Scheduler

为什么Linux CFS调度器没有带来惊艳的碾压效果
[Linux桌面GUI系统的调度器应该怎么做才不卡顿呢？](https://blog.csdn.net/dog250/article/details/96500186)
[为什么Windows/iOS操作很流畅而Linux/Android却很卡顿呢](https://blog.csdn.net/dog250/article/details/96362789) dog250这位博主在CSDN上从2009年开始，一直在写，文章还不错（不要因为CSDN就不看啊，这是早期良心用户）。这样的稀有博主是我一直没舍得狠下心屏蔽CSDN的原因（颇有一种在垃圾坑里淘宝的感觉）

[调度系统设计精要 - 面向信仰编程](https://draveness.me/system-design-scheduler/)

http://ck.kolivas.org/

https://liquorix.net/ 这个内核是基于zen内核加了一些其他的补丁和改动（有ck补丁，但是没有MuQSS，原因`I'm severely prejudiced against MuQSS, so it will continue "giving a null".`, 见这里https://bugs.archlinux.org/task/56312）

[The Linux Scheduler: a Decade of Wasted Cores](https://people.ece.ubc.ca/sasha/papers/eurosys16-final29.pdf) 讲多核的

https://blog.ihypo.net/15279557709685.html

http://linuxperf.com/?p=42

<!-- https://blog.csdn.net/XD_hebuters/article/details/79623130

https://cloud.tencent.com/developer/article/1603970

https://cloud.tencent.com/developer/article/1603974 -->

https://www.cnblogs.com/hellokitty2/p/14199741.html

https://github.com/theanarkh/read-linux-0.11

http://www.wowotech.net/process_management/449.html

[Linux进程调度-组调度及带宽控制](https://www.cnblogs.com/LoyenWang/p/12459000.html)

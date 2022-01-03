---
title: 番外：Linux内核调试
date: 2021-07-04 17:10:09
tags:
- Linux
- kernel
- debug
- 6.S081
abbrlink: kernel-debugging
categories:
- Linux&Unix
---
之前写那个Linux抢救和维护的时候提过一下，现在做6.S081的Lab又碰到的这个问题，单独开一篇记录一下（水平不够，调试来凑）
<!-- more -->

## printk()

如果内核的错误还没有严重到来不及输出错误信息，还是可以靠print大法调试的。

printk输出到日志的内容可以用`dmesg`，普通的驱动问题靠输出的log定位问题就可以了。比如我要找HDMI的log：`dmesg | grep HDMI`。如果信息还不够多就加一个参数`CONFIG_DEBUG_DRIVER`到[kernel parameter](https://wiki.archlinux.org/title/Kernel_parameters)里面，这会把来自Linux Kernel Driver Database的原始数据全都打印出来（非常非常多，除非写驱动的或者Debug找不到足够信息否则不要开）

printk有7个级别，见[Message logging with printk](https://www.kernel.org/doc/html/v5.12-rc3/core-api/printk-basics.html)。从0-7打印信息逐渐增加，调试的时候选一个合适的就行了

```c
printk(KERN_WARNING "这里好像有问题，打印一下");
```

### Kernel Panic 后来不及printk

printk是异步的
`printk.synchronous=1`

printk()有一个问题，每次想添加printk()的时候都要重新编译（6.S081/6.828用的教学xv6那种几千行代码的玩具操作系统还好办，Linux内核这样庞大的东西CPU不太好的时候要编译很久的）

## /proc/sys等目录

这几个目录比较重要，临时修改参数可以用

## 其他工具

systemtap，相当于一个kprobe的封装

## qemu等模拟器/虚拟机

虚拟机里面比较简单，有很多方便的方案可以将日志、页表、堆栈调用给你打印出来。

### gdb+qemu

下面以6.S081/6.828为例：

按照课程的推荐装一个合适的gdb（gdb要支持相应的架构，），比如我要调试的内核是RISC-Ⅴ的，在Archlinux下应该是`riscv64-linux-gnu-gdb`, 在Ubuntu20.04下可以用`gdb-multiarch`。

然后打开两个terminal，第一个执行`make qemu-gdb`启动qemu模拟器， 第二个执行gdb`gdb-multiarch`。

这样就可以愉快的用gdb调试了，具体操作见另一篇。（太长不看版，就是普通程序gdb怎么调试这里就怎么调呗）

## 其他环境

### 物理机（比如Linux桌面发行版）

这种比qemu模拟器里面的要麻烦很多，你可能来不及看到错误log就崩溃了，得借助kdump、crash等工具把崩溃信息记下来。

如果能进grub，

### 开发板或裸机

ARM的板子就直接用Jlink呗。以前写的烧到板子上的东西都是不超过两千行的小玩意，没啥调试经验，不讲了。（当时全靠dmesg）

## 调试的原理

这些内容由于我实在太菜，写不动了。写的不好，将就着看吧。稍微了解一下原理对使用这些调试工具有一些帮助

### gdb

gdb的实现利用了ptrace的systemcall

### printk 

直接看这两篇文章吧：
[printk: Why is it so complicated?](https://www.linuxplumbersconf.org/event/4/contributions/290/attachments/276/463/lpc2019_jogness_printk.pdf) Linux Plumbers Conf 的ppt  
[Why printk() is so complicated (and how to fix it)](https://lwn.net/Articles/800946/)  

### systemtap


## 你真的需要Debugger吗

Linus在邮件[Re: Availability of kdb](https://lwn.net/2000/0914/a/lt-debugger.php3)里是这样说的：`
I don't like debuggers. Never have, probably never will. I use gdb all the
time, but I tend to use it not as a debugger, but as a disassembler on
steroids that you can program.`

这封邮件比较长，截取一段代表性观点

> I happen to believe that not having a kernel debugger forces people to think about their problem on a different level than with a debugger. I think that without a debugger, you don't get into that mindset where you know how it behaves, and then you fix it from there. Without a debugger,you tend to think about problems another way. You want to understand things on a different _level_.

系统编程不像平时糊个网站搓个脚本那样，不能过于依赖Debug，应该一开始就设计好，出bug说明设计有问题、写的人不仔细。把事情都想清楚写好了编译一次通过，这才是Linus认为的合格的(水平要达到不依赖debugger)

>you start being careful, or you start whining about a kernel debugger.

>Quite frankly, I'd rather weed out the people who don't start being
careful early rather than late. 

老派程序员里面很多经验丰富的大牛都不用debugger, 一种是习惯了print，另外很多典型的观点是Robe Pike这样的

>If you dive into the bug, you tend to fix the local issue in the code, but if you think about the bug first, how the bug came to be, you often find and correct a higher-level problem in the code that will improve the design and prevent further bugs.

我试着不用gdb调试、语法高亮和代码补全做了6.S081的Lab1，结果相当惨烈。这几天回去翻出了藏在柜子里面的k&r，从图书馆拿来了《c陷阱和指针》和《debug hacks》，并给VSCode启用了clangd自动补全并配置了断点调试。（我甚至想掏出clion了）

如果你像我一样菜，去找/买个大屏幕（最好能竖过来），然后打开你的debugger
（强烈推荐4k显示器，画质飞跃，头也不疼眼也不花了）

（据说一个大的显示屏有buff加成，据说，据说再在旁边摆一个小显示器开个终端也不错）

当然，有的时候你也只是想找一下驱动的bug并汇报给厂家，或者你只是个普通用户，想检查一下内核更新后开不了机的问题；掏出调试工具吧！

## 链接

《Debug Hacks 深入调试的技术和工具》 （吉冈弘隆） 这本书讲的很好，这还是之前我在图书馆乱逛无意间翻到的，结果发现是讲内核的，当时根本看不懂。今天又从图书馆翻出来了，发现讲的还挺好。


https://lwn.net/2000/0914/a/lt-debugger.php3  
https://www.kernel.org/doc/html/v5.12-rc3/core-api/printk-basics.html  


[printk: Why is it so complicated?](https://www.linuxplumbersconf.org/event/4/contributions/290/attachments/276/463/lpc2019_jogness_printk.pdf) Linux Plumbers Conf 的ppt  
[Why printk() is so complicated (and how to fix it)](https://lwn.net/Articles/800946/)  
[内核日志：API及其实现](https://web.archive.org/web/20120325222504/http://www.ibm.com/developerworks/cn/linux/l-kernel-logging-apis/index.html)  

https://elinux.org/Debugging_by_printing  

另外[Embedded Linux Wiki](https://elinux.org/Main_Page)里面挺多东西都挺好的（你不搞嵌入式那当我没说）

https://blog.csdn.net/tangkegagalikaiwu/article/details/8572365
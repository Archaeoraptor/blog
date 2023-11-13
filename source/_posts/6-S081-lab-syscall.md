---
title: 6.S081 Lab Syscall 笔记
date: 2021-09-15 20:01:25
abbrlink: '6.s081-lab-syscall'
tags:
- 6.S081
- syscall
categories:
- Linux&Unix
---
暑假摸了一个月的鱼，回来接着做Lab。2021 spring 的6.S081已经开始了，直接做2021的吧（看了一下好像没有太大变化）。而且xv6-labs-2021这个repo貌似修复了Archlinux上新版gdb不兼容的问题，不用再降级包或者去用Ubuntu 20了。
<!-- more -->

注：目前现在2021的repo由于进度原因只有riscv、util和syscall三个分支，没有master分支，所以会报警告：`warning: remote HEAD refers to nonexistent ref, unable to checkout.`这个不用管。

## 关于syscall

syscall一般很少直接用，用c语言写东西的时候大部分都是通过wrapper函数间接调用syscall。其他语言里用到系统调用的就更少了，我就只在某次用golang的网络库实现traceroute的时候调用过一次。

之前对syscall的印象是x86下`int 80`中断，从用户空间切到内核空间。常用的比如read, write, open, close啊，以及mmap等等。

6.S081改用risc-v, 不太一样。
risc-v中有三种模式：
User-mode，用户模式。普通应用会在这个模式下运行。
Supervisor-mode， 特权模式。
Machine-mode，机器模式，需要的权限比supervisor模式还要高。一些嵌入式几乎整个都运行在这种模式下，完整的操作系统就只有bootloader等是在M模式下，大部分任务都在U和S模式下。  
关于risc-v的特权模式可以参考手册：The RISC-V Instruction Set Manual
Volume II: Privileged Architecture

进入syscall是通过ecall（环境调用）指令，从u模式进入s模式，结束后使用`sret`指令返回。

还有就是讲义中将异常（exception）和中断（interrupt）都算做陷阱（trap），一开始看讲义的时候一直以为这是三个不同的东西。后来发现涉及到supervisor和machine模式的都算做trap。

## Lab2

Lab2就两个，简单熟悉一下系统调用，不难。

### System call tracing (moderate)

这个不是很难，不过要改的东西很多，按照提示一点一点做就行了。

## Linux下的syscall

这里说一下linux中系统调用相关的东西，6.S081的课和Lab没怎么提到，但是这一部分写业务用到的比较多。linux下的syscall一般直接用glibc提供的，现在一般x86下也不用`int 0x80`了。由于syscall的调用消耗很高，所以有vsyscall和vDSO等技术。

以date命令为例，我们用strace看一下系统调用，可以看到clock_gettime调用花了0.000006s

```bash
$strace -cT
strace: -T/--syscall-times has no effect with -c/--summary-only
Tue Mar 22 07:16:09 PM CST 2022
% time     seconds  usecs/call     calls    errors syscall
------ ----------- ----------- --------- --------- ----------------
 22.18    0.000057           6         9           mmap
 12.45    0.000032           5         6           close
 12.45    0.000032           5         6           newfstatat
 11.67    0.000030           7         4           mprotect
  7.00    0.000018           4         4           openat
  4.28    0.000011           3         3           read
  4.28    0.000011          11         1           munmap
  4.28    0.000011           3         3           brk
  2.72    0.000007           7         1           write
  2.33    0.000006           1         4           pread64
  2.33    0.000006           6         1           clock_gettime
  2.33    0.000006           6         1           getrandom
  1.95    0.000005           5         1           lseek
  1.95    0.000005           2         2         1 arch_prctl
  1.95    0.000005           5         1           set_tid_address
  1.95    0.000005           5         1           set_robust_list
  1.95    0.000005           5         1           prlimit64
  1.95    0.000005           5         1           rseq
  0.00    0.000000           0         1         1 access
  0.00    0.000000           0         1           execve
------ ----------- ----------- --------- --------- ----------------
100.00    0.000257           4        52         2 total
```

vsyscall是将频繁使用的syscall从kernelsapce映射到userspace的页，这样就不需要切换usersapce和kernelspace了。这样对`gettimeofday`这样的对时间很敏感的系统调用好处很大。但是vsyscall将kernelspace一段固定的地址映射到userspace，存在安全问题。现在已经不常用vsyscall了，基本都改用vDSO了。

vDSO（virtual dynamic shared object）是一个虚拟的动态库(linux-vdso.so.1)，同样运行在userspace，这样就不需要软中断了。vDSO一个好处是利用ASLR（address space layout randomization）减少了vsyscall的安全问题。现在linux发行版一般都开了地址随机化（可以看`/proc/sys/kernel/randomize_va_space`），调试内核的时候我建议临时把这个关掉。另一个好处是把不同平台的syscall封装了，给了一套统一的ABI，动态链接库的ABI从kernel的libc独立出去了。

另一个加快系统调用的技术是快速系统调用（fast system call），这个是x86的一些  指令，直接切换特权，也不需要中断。 

## 链接

<https://man7.org/linux/man-pages/man2/syscalls.2.html>  
<https://filippo.io/linux-syscall-table/>  


[Anatomy of a system call, part 1](https://lwn.net/Articles/604287/)    
[Anatomy of a system call, part 2](https://lwn.net/Articles/604515/)  
[On vsyscalls and the vDSO](https://lwn.net/Articles/446528/) 这个文章有点老，不感兴趣不用看
[The Definitive Guide to Linux System Calls](https://blog.packagecloud.io/the-definitive-guide-to-linux-system-calls/) 相当详细的一片文章，强烈推荐  
[[译] Linux 系统调用权威指南（2016）](https://arthurchiao.art/blog/system-call-definitive-guide-zh/#1-系统调用是什么) 上面那篇的中文翻译  

[linux-insides-zh](https://github.com/MintCN/linux-insides-zh) 中文翻译  
[x86 架构下 Linux 的系统调用与 vsyscall, vDSO](https://vvl.me/2019/06/linux-syscall-and-vsyscall-vdso-in-x86/)  
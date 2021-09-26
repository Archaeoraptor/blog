---
title: 6.S081 Lab Syscall 笔记
date: 2021-09-15 20:01:25
tags:
- 6.S081
- syscalls
categories:
- 笔记&札记
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

进入syscall是通过ecall（环境调用）指令，大部分都是从u模式进入s模式，结束后使用`sret`指令返回。

还有就是讲义中将异常（exception）和中断（interrupt）都算做陷阱（trap），一开始看讲义的时候一直以为这是三个不同的东西。后来发现涉及到supervisor和machine模式的都算做trap。

## Lab2

Lab2就两个，简单熟悉一下系统调用，不难。

### System call tracing (moderate)

这个不是很难，不过要改的东西很多，按照提示一点一点做就行了。先添加

## 链接

https://man7.org/linux/man-pages/man2/syscalls.2.html
https://filippo.io/linux-syscall-table/  


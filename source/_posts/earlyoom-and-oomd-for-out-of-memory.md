---
title: Earlyoom和oomd使用，解决Linux内存耗尽的一点尝试
date: 2021-09-14 16:08:11
tags:
- earlyoom
- oomd
- OOM
---
太长不看：就是把原来内核的OOM killer挪到用户空间（userspace）去，这样就可以在系统卡住之前（通常桌面卡住的时候OOM killer认为还不用kill）提前kill掉占内存最大的一个，让桌面和鼠标可以正常工作。
<!-- more -->

## 前言

先来看phoronix的一个报道：[Yes, Linux Does Bad In Low RAM / Memory Pressure Situations On The Desktop](https://www.phoronix.com/scan.php?page=news_item&px=Linux-Does-Bad-Low-RAM)  

Linux桌面在小内存、内存紧张的情况下表现很差。（用过Linux现在桌面的相信都有这种感受）

解决内存不足、内存耗尽的**最好办法还是加钱多装几根内存条**，什么swap、zram还有本文的earlyoom和oomd都是权宜之计。 

## OOM简介

OOM很多人应该都很熟悉了，服务器上跑的任务申请内存超过了可用内存就会发生OOM（其实是可以Overcommit的，这个以后再说）

为了避免内存耗尽（Out of Memory，OOM），OOM-killer在内存将要耗尽的时候杀掉`oom_score`最大的进程。（如果连OOM-killer都没能及时kill掉，那就会触发kernel panic）

但是内核默认的OOM策略比较保守，如果你是桌面用户，那么还没等到OOM killer工作，图形界面就已经卡死（handling， 有的时候也被称为freeze）了。Earlyoom和oomd等工具运行在用户空间，让一些任务提前崩溃，让图形界面还能正常工作。

OOM killer和Overcommit等更详细的介绍我写到[这里](https://zhangjk98.xyz/linux-out-of-memory), 这里不多说了。

## Earlyoom

EarlyOOM

Earlyoom在桌面系统的表现比较成功，Fedora 32 已经默认启用了EarlyOOM。

## oomds

oomd是Facebook做的，后来和systemd一起做了一个sysytemd-oomd守护进程，现已加入systemd全家桶。

## 在Archlinux上安装和使用

Earlyoom已经在community源里了。oomd这个包目前是orphand，我们用Earlyoom就好了。

```bash
pacman -S earlyoom
```

直接terminal里执行`earlyoom`就可以了。（可能需要root权限，不然会报`Could not lock memory - continuing anyway: Cannot allocate memory`）

或者使用systemd守护进程

```bash
systemctl enable earlyoom --now
```

然后又到了调参环节，编辑`/etc/default/earlyoom`，根据你的机器配置和使用习惯改改参数。

默认10秒检查一次，当可用内存小于10%和swap小于10的时候执行`SIGTERM`，当可用内存小于5%和swap小于5%的时候执行`SIGKILL`。

## 经历

一开始我笔记本和台式机都8G内存，当时那个版本的VSCode有严重的内存泄漏，经常泄漏到我桌面连鼠标都动不了了。只能切到tty杀掉VSCode的进程或者魔术键重启，烦的我一度想投奔vim。  
后来买了两根16G的笔记本内存条，又把教研室的台式内存（拆了4根8G ddr3的杂牌内存条下来）加到了32G，开着一个虚拟机好多Docker一个Goland还有Chrome，再也没遇到过内存不够用的问题。 
最近要在另一个8G内存的电脑上和只有2g内存的VPS上面干点活，又回到了当初只有8G内存的时候vscode频繁内存泄漏导致整个桌面直接卡死的拮据状态。Ubuntu 16.04 那个版本的GNOME内存泄漏和内部错误也相当讨厌。换了lxde好了一点。然后试了试swap，效果不好。然后用了zram, 效果比swap好不少。  

最后用了Earlyoom，好多了，不用频繁手动处理桌面卡死了。（注意4G以下这种小内存不推荐用这些，会浪费不少可用内存）

## 链接

https://github.com/rfjakob/earlyoom  

Fedora默认启用Earlyoom的一些讨论：

https://fedoraproject.org/wiki/Changes/EnableEarlyoom#Enable_EarlyOOM  
https://pagure.io/fedora-workstation/issue/119  
https://www.phoronix.com/scan.php?page=news_item&px=Fedora-32-Default-EarlyOOM  

[Linux vm运行参数之（二）：OOM相关的参数](http://www.wowotech.net/memory_management/oom.html)



---
title: 现代文件系统笔记，顺便说说Btrfs
tags:
- btrfs
- fs
abbrlink: 'filesystem-note-btrfs-introduction'
categories:
- Linux&Unix
---

![](https://imgs.xkcd.com/comics/dark_arts.png)
之前做6.s081的lab的时候照猫画虎写了一个类似ext2的超简陋的文件系统，自己用了有将近一年的Btrfs，一直想写点什么。然而之前看OS的书和做Lab都是
<!-- more -->

## 从简单的文件系统说起

我们先不管日志、快照、原子性、加密这些功能。也暂时先不看类似9p那种分布式文件系统。
磁盘被划分成很多块（block），我们拿出来一些块放用户数据（data region），为了记录文件的

## 现代文件系统

>如果reiserfs没有因为作者杀妻进去
>如果SUN没有被Oracle收购，ZFS没那些版权问题


### Reference Link



## 说说Btrfs

Btrfs推出也很久了，最初是打算取代ext3/ext4地位做默认的新一代文件系统的。当然，Btrfs已经十几年了，进入Linux内核主线都8年了，大家也看到了。。。

最近的Linux内核每次更新几乎都有Btrfs的内容，而且在phoronix之类的地方讨论总是很热烈

[](https://www.phoronix.com/forums/forum/software/general-linux-open-source/1276327-btrfs-adds-degenerate-raid-support-performance-improvements-with-linux-5-15)  
[There's A Proposal To Switch Fedora 33 On The Desktop To Using Btrfs](https://www.phoronix.com/scan.php?page=news_item&px=Fedora33-Desktop-Btrfs-Proposal)  

这话怎么听着这么耳熟，隔壁Wayland是不是好多年前也说过要*&&%&*￥%*&……，真是不巧我还在用x11呢。。。

现在在用Btrfs的大概也就Facebook，群晖，默认用Btrfs的发行版几乎就openSUSE，然后去年好像Fedora也把Btrfs作为默认了（可能）。

### 关于Btrfs性能不好

Btrfs性能不好的说法主要来源于跑分跑不过ext4等等。跑分这种事情很多时候也就图一乐，以下引用fc老师的话

>phoronix 測文件系統性能的最大問題在於容易讓讀者以為文件系統是左右存儲性能的關鍵，實際上文件系統只是夾在內存子系統和塊IO子系統中間的決策層，通常瓶頸不會在文件系統，除非文件系統過度簡化設計，對存儲性能影響更大的是別的層面的東西，按他們的方式測試文件系統只能測出一堆難以預計到的副作用的結果，沒法對文件系統優化提供可供參考的提示

>要測純讀寫做對比肯定不考慮 reflink 和透明壓縮，但是實際幹活的時候這些新特性很節省時間（

当然Btrfs在重io的情况下桌面特别卡，那很有可能是quota的问题，把quota关掉吧，quota有已知的性能问题，尤其是在ssd上。（关了这个磁盘配额你就看不到快照大小、指定不了每个子卷的大小）

### Copy-on-Write写时复制

写时复制（COW）原理看起来好像很简单，但是实现起来一堆坑。我做6.S081的Lab的时候折腾COW快把我整麻了，从Lab4以后每次做完Lab之后不想写博客总结了，做完之后思绪混乱，没什么心情写笔记。

#### 给数据库所在的目录关闭COW

SSD会有写入放大的问题，这个现象在Btrfs上尤其严重。
不想和Btrfs和数据库斗智斗勇可以选择别的文件系统。

### 使用

如果用不到现代文件系统的那些功能，对于大多数用户直接去用ext4可能是一个更好的选择。然后忘了文件系统这回事吧，就像开头xkcd的漫画说的那样：`BUT I NOW `

### 都要2022年了，Btrfs稳定了吗

无可奉告。
平时用倒是能用，不过真出了问题

### Btrfs挂了要怎么修

~~去群里找fc老师~~

做好备份，真挂了把备份找回来。大多数修复软件几乎都不怎么支持Btrfs，你去电脑城找做修复的师傅人家也不会，自求多福吧。

### RAID、子卷、分区问题

之前用ext4等文件系统很多都习惯`/`，`/home`, `/var`
Btrfs一般不分那么多区，而是用子卷。

一般对`var`这样的目录单独建一个子卷，然后禁用COW

Btrfs可以直接添加、删除设备、调整大小（增加和缩小都行，XFS暂时还不支持缩）

### snapshot

#### 要怎么查看snapshot的大小

查看snapshot大小需要打开quota，但是打开quota可能会有性能问题。

## 链接

[Analyzing IO Amplification in Linux File Systems](https://arxiv.org/pdf/1707.08514.pdf)  
[XFS: There and back ... and there again?](https://lwn.net/Articles/638546/)  
[Btrfs vs ZFS 实现 snapshot 的差异](https://farseerfc.me/zhs/btrfs-vs-zfs-difference-in-implementing-snapshots.html)  
[Выявляем процессы с дисковой активностью в Linux](https://habr.com/ru/post/476414/)  一篇俄文的Write Amplification讨论  
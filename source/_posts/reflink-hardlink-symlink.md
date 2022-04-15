---
title: 文件系统的Reflink
date: 2021-12-19 13:24:54
tags:
- fs
- reflink
abbrlink: 'reflink-hardlink-symlink'
---
reflink暂且翻译成引用链接吧，新版coreutils的cp和mv的默认行为就是reflink（如果文件系统支持）。硬链接和软链接的
<!-- more -->

## Reflink

如果你的coreutils>9.0，而且装的是Btrfs或XFS等支持Reflink的文件系统，那么`cp`和`mv`的默认行为是使用reflink(mv应该是在不同子卷之间，when moving files across BTRFS subvols)，`cp --reflink=auto`

最早是从Btrfs听说的reflink,以为这就是一个COW，后来发现它并不一定需要COW，而是和de-duplication（重复数据删除）有关。

## 整理碎片对reflink的影响

以Btrfs为例，defragment会打断reflink，于是你做完defrag以后会发现占用空间可能大了好多。不过像Btrfs之类的文件系统几乎都跑在SSD上，整理碎片不仅不会像机械那样提升性能，而且会增加写入。所以直接不整理碎片就好了。

## 硬链接和软链接和Reflink的区别

这两个东西的区别基本上网上都说烂了。这里再简单说一下。

硬链接（hardlink）的inode号和原来的一样

但是hardlink和symlink有一些问题，创建的时候很开心，修改和删除就比较麻烦。

## Reflink这个东西有啥用

1.加快复制和移动速度
2.节省空间

比如wine的reflink补丁。比如你steam开proton下了很多游戏，你会发现`.local/share/Steam`下面每个游戏的目录都有一堆相同的ddl。reflink的补丁可以有效减小Wine应用的体积，很多重复的ddl文件都会以reflink的方式复制。

[Proposed Reflink Support Would Provide Big Space Savings For Wine](https://www.phoronix.com/scan.php?page=news_item&px=Reflink-For-Wine-Patches)， [Wine Reflink Support Continues To Be Worked On For Significant Space Savings](https://www.phoronix.com/scan.php?page=news_item&px=Wine-Reflink-Revised)  

坏消息这玩意暂时不支持ext4，好消息Steam Deck将采用Btrfs。对于不支持的文件系统，可以通过OVerlayfs实现，不过好像没多少人这么干（Overlayfs这个名字你也在Docker那里听说过对罢？坑也不少）

那wine为什么不用符号链接呢？看起来这种情况下用symlink不是也挺合适的？

>Because most of the file WINE is running didn’t have symlink support in mind.They assumes that the open give them the file, not a symlink.After opening the file, they would just update it without replacing that symlink with an actual file.

另一个麻烦的问题是相当多的游戏反作弊软件，他们在检查的时候不认符号链接。于是就认为你这ddl有问题，然后就封号了。用reflink就比较好解决了，在Wine应用和你看来是感觉不到链接存在的，reflink看起来就像是正常的文件一样；而symlink看起来就是一个link，如果Wine应用和外挂检测不去处理他会出问题。举一个不恰当的例子，reflink就像透明代理一样，用户和软件感觉不到它的存在。

要不我们叫它透明链接算了（逃

## 链接

https://blogs.oracle.com/linux/post/xfs-data-block-sharing-reflink  
[Deduplication Btrfs Wiki](https://btrfs.wiki.kernel.org/index.php/Deduplication)  
[Should You Defrag an SSD? ](https://www.crucial.com/articles/about-ssd/should-you-defrag-an-ssd) 

https://man7.org/linux/man-pages/man2/ioctl_ficlone.2.html

https://lwn.net/Articles/331808/
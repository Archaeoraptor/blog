---
title: 6.S081 Lab mmap 笔记
date: 2022-04-15 19:17:29
tags:
- 6.S081
- mmap
abbrlink: '6-S081-lab-mmap'
---
mmap实现细节坑略多。本来想再写点linux的sendfile和splice相关的内容的，但是源码比较麻烦，算了，先大概聊聊原理吧
<!-- more -->

如果不在乎性能，那么文件的读写只要read和write这样的syscall就可以了，mmap、sendfile、slice等技术都是为了性能，通过某些方式减少系统调用和拷贝。

read和write作为系统调用是很慢的，大概要经过磁盘数据->内核page cache

### mmap

这个大家都很熟悉吧，最早的也是最经典的。申请内存的时候经常用mmap。思路也比较简单，将文件映射到内存里，然后像读写文件一样读写内存。mmap将文件映射到了进程的虚拟地址空间（vma），操作内存可以达到直接操作块设备的效果。（注意这里和直接将文件放在物理内存里是不一样的）

我之前有一个误解是mmap的开销相比read、write小很多，后来发现mmap的开销包括

### sendfile

### splice

splice通过管道在两个socket之间传数据

## Lab


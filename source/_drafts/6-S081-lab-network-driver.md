---
title: 6.S081 Lab Networking 笔记
date: 2022-04-15 17:15:11
tags:
- 6.S081
- network
abbrlink: '6-S081-lab-network-driver'
---
这个Lab就是补全一个DMA收发的驱动, 不要被Lab标的Hard难度吓到，我感觉这是6.S081这几个Lab最简单的几个Lab之一。写个驱动都不用上板子调试，这也太水了。
<!-- more -->

>Your job is to complete e1000_transmit() and e1000_recv(), both in kernel/e1000.c, so that the driver can transmit and receive packets

这个好办，我们直接扒拉e1000的手册就可以了

## 关于驱动和网络

又是讨厌的驱动，勾起了我当年痛苦的。上一次写驱动的大作业还是本科的时候，当时换了两块xilinx的阴间开发板全都是坏的，害得我还以为是我自己写的有问题，找了一周没找到bug。

好久之前写的Lab，由于懒癌发作一直没有写笔记，现在把Lab的笔记补上。这个Lab比较玩具，没啥好讲的。驱动和网络单独拿出来都是一个大坑，我水平有限，不多说了。

这个Lab的Optional Challenges比较麻烦，其中最后两个challenge是：

>Implement a UDP server for xv6. (moderate)
>Implement a minimal TCP stack and download a web page. (hard)

由于懒癌发作我当然又是没做，我的菜鸡计网水平和对TCP的熟悉程度应该一时半会做不出来。听说有cs144这门课就是实现一个TCP协议栈，等研三有空了回头把这个做了把这个大坑填上。
---
title: 6.S081 Lab Networking 笔记，完结撒花
tags:
  - 6.S081
  - network
abbrlink: 6-S081-lab-network-driver
date: 2022-05-05 17:15:11
categories:
- Linux&Unix
---

这个Lab就是补全一个DMA收发的驱动, 不要被Lab标的Hard难度吓到，我感觉这是6.S081这几个Lab最简单的几个Lab之一。写个驱动都不用上板子调试，没啥意思，怎么感觉比当年你电PPT吹水课程微嵌的实验大作业给FPGA的板子写个驱动还水。。。  
最后一个Lab放水，愉快的完结撒花。也难怪在2021 fall的lab net不是最后一个。等等，这个Lab还有附加内容，实现一个TCP/UDP协议栈。。。
<!-- more -->

## Lab内容

这个难度明明是easy，不要被标的那个hard吓到了。做起来这个Lab挺没意思的，e1000这种远古网卡，现在应该没多少人用了。这个网卡是qemu虚拟出来的网卡，就跟代理的时候用的那个tun/tap差不多

>Your job is to complete e1000_transmit() and e1000_recv(), both in kernel/e1000.c, so that the driver can transmit and receive packets

这个好办，我们直接扒拉e1000的手册就可以了。或者我们直接看`kernel/e1000_dev.h`, `E1000_RDH`是E1000 RX Descriptor Head， `E1000_RDT`是RX descriptor tail，同理`E1000_TDT`是TX descriptor tail（由TX的功能我们可以知道这是the next packet的坐标）， `E1000_RXD_STAT_DD`是 Descriptor Done， 然后`E1000_TXD_CMD_EOP`是 End of Packet。

RX ring接受网卡数据的读取buffer， TX ring是发送packet的发送buffer。`rx_ring`是一个环形链表`rx_desc`，里面放的是packet的地址（packet是通过DMA放在内存里面）

```c
// [E1000 3.2.3]
struct rx_desc
{
  uint64 addr;       /* Address of the descriptor's data buffer */
  uint16 length;     /* Length of data DMAed into data buffer */
  uint16 csum;       /* Packet checksum */
  uint8 status;      /* Descriptor status */
  uint8 errors;      /* Descriptor Errors */
  uint16 special;
};
```

然后我们照着提示就可以补全`kernel/e1000.c`了

```c
int e1000_transmit(struct mbuf *m) {
    // the mbuf contains an ethernet frame; program it into
    // the TX descriptor ring so that the e1000 sends it. Stash
    // a pointer so that it can be freed after sending.
    //
    acquire(&e1000_lock); // 发送时加spin_lock
    uint32 idx = regs[E1000_TDT];
    if (!(E1000_TXD_STAT_DD & tx_ring[idx].status)) { // buffer不足
        release(&e1000_lock);                         // 释放锁
        return -1;
    }
    if (tx_mbufs[idx]) {
        mbuffree(tx_mbufs[idx]);
        tx_mbufs[idx] = 0;
    }

    tx_ring[idx].addr = (uint64)m->head;
    tx_ring[idx].length = m->len;

    tx_ring[idx].cmd = E1000_TXD_CMD_EOP | E1000_TXD_CMD_RS;
    tx_mbufs[idx] = m;

    regs[E1000_TDT] = (idx + 1) % TX_RING_SIZE;
    release(&e1000_lock);

    return 0;
}

static void e1000_recv(void) {
    //
    // Your code here.
    //
    // Check for packets that have arrived from the e1000
    // Create and deliver an mbuf for each packet (using net_rx()).
    //
    while (1) {
        // First ask the E1000 for the ring index at which the next waiting
        // received packet (if any) is located, by fetching the E1000_RDT
        // control register and adding one modulo RX_RING_SIZE.
        uint32 idx = regs[E1000_RDT];
        idx = (idx + 1) % RX_RING_SIZE; // 这是个环形链表，需要mod ring size
        if (!(E1000_RXD_STAT_DD & rx_ring[idx].status)) {
            return;
        }
        rx_mbufs[idx]->len = rx_ring[idx].length;
        net_rx(rx_mbufs[idx]);
        rx_mbufs[idx] = mbufalloc(0); //这里直接mbufalloc就可以了，不需要kalloc
        rx_ring[idx].addr = (uint64)rx_mbufs[idx]->head;
        rx_ring[idx].status = 0;
        regs[E1000_RDT] = idx;
    }
}
```

## 完结撒花

```bash
$ make grade
make[1]: Leaving directory '/home/zhixi/codes/xv6-labs-2021'
== Test running nettests == 
$ make qemu-gdb
(2.8s) 
== Test   nettest: ping == 
  nettest: ping: OK 
== Test   nettest: single process == 
  nettest: single process: OK 
== Test   nettest: multi-process == 
  nettest: multi-process: OK 
== Test   nettest: DNS == 
  nettest: DNS: OK 
== Test time == 
time: OK 
Score: 100/100
```

一开始以为很难，又是讨厌的驱动，然后一看难度还是hard，勾起了我当年痛苦的。上一次写驱动的大作业还是本科的时候，当时换了两块xilinx的阴间开发板全都是坏的，害得我还以为是我自己写的有问题，找了一周没找到bug。本来是抱着人又麻了的心态来做这个Lab的（这个Lab在2020年是最后一个Lab，在2021年放到了前面），拖了几个月，别的Lab都做完了，然后陆续把笔记也补上了，最后捡起了一直咕咕咕的6.824，做到Raft的Lab4死活跑不过测试这才回来做这个Lab，结果发现意外的简单。

好久之前写的Lab，由于懒癌发作一直没有写笔记，现在把Lab的笔记补上。这个Lab比较玩具，没啥好讲的。驱动和网络单独拿出来都是一个大坑，水平有限不多讲了。

这个Lab的Optional Challenges比较麻烦，其中最后两个challenge是：

>Implement a UDP server for xv6. (moderate)
>Implement a minimal TCP stack and download a web page. (hard)

然后搜了一下发现有一个xv6-net的实现：<https://github.com/pandax381/xv6-net/blob/net/tcp.c>

由于懒癌发作我当然又是没做，我的菜鸡计网水平和对TCP的熟悉程度应该一时半会做不出来。听说有cs144这门课就是实现一个TCP协议栈，等研三有空了回头把这个做了把这个大坑填上。

6.S081这个课从去年7月开始，断断续续做了将近一年把Lab做了，xv6的源码看了不到一半。xv6作为一个教学的系统，挺精简挺小巧，能比较顺利的看下去。像linux源码那种看不太动，拿着赵炯那本《linux内核完全注释》看早期内核也比较麻烦。xv6-book写的真的好，推荐做Lab和读xv6源码的时候好好把xv6-book看一下：

*xv6: a simple, Unix-like teaching operating system*
作者Russ Cox, Frans Kaashoek, Robert Morris 应该都听过，Russ Cox是争议比较大的go现任掌门，喜欢在go的issue区开喷，Morris是6.824这门课的主讲，就是那个蠕虫病毒的作者。

一开始我没在意，觉得这本书就是跟手册一样，偶尔翻一翻。后来发现这本册子写的是真的好，几乎没有一点废话，比课程讲义好多了。早应该看这个的。

然后就是重新翻了k&r这本书，写的真好，也没有多少废话。里面的例子非常精简。

更新：不等研三了，直接上CS144了。之前没事喜欢摆弄nmap、wireshark这些东西瞎玩，却没好好看过TCP的细节，再这样不思进取下去就要沦为脚本小子了
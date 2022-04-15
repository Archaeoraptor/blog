---
title: 浅谈SSD写入放大
date: 2022-04-14 18:46:12
tags:
- SSD
- Write-Amplification
- LSM
abbrlink: 'write-amplification'
---
纪念上一块编译报废的固态。
<!-- more -->

## 为什么会有写入放大

### gc造成的写入放大

SSD的写入是先擦除再写入（Erase-Before-Write）

## LSM Tree

李世民树，是李世民体恤SSD读写太高耗费国库所有的树 (大雾



## 说一下Btrfs写入放大的问题


我之前以为

关于Btrfs文件系统数据库要关闭COW的问题，直接贴一下fc老师的话

>虛擬機鏡像，InnoDB 那種底層是 b-tree 結構的數據庫（leveldb/rocksdb/myrocks 那種底層是 lsm tree 的不需要
>因為這些 b-tree 數據庫自己做 4k/8k/16k 規模的 b-tree node 隨機讀寫，隨機讀寫和 cow fs 相性不好，尤其開了 compress 的 btrfs 每個 compressed extent 可以到 128K 就會有 32 倍的寫放大。（順便這種讀寫方式和有 ftl 的 ssd ，有 stl 的 smr 的相性也不好…

## 利用内存减少SSD读写



## 链接

[Ask HN: Which Linux filesystem “produce” less wear and tear on SSD NAND?](https://news.ycombinator.com/item?id=22767106)  
[Выявляем процессы с дисковой активностью в Linux](https://habr.com/ru/post/476414/)  一篇Write Amplification讨论  

[](A Comparison of Fractal Trees to Log-Structured Merge (LSM) Trees)
[Write amplification analysis in flash-based solid state drives](https://dl.acm.org/doi/10.1145/1534530.1534544)  
[Thread: Leap & TW: Enormously high write amplification with BTRFS](https://forums.opensuse.org/showthread.php/548476-Leap-amp-TW-Enormously-high-write-amplification-with-BTRFS)  


[ArchWiki Solid state drive](https://wiki.archlinux.org/title/Solid_state_drive)  
[ArchWiki Tmpfs](https://wiki.archlinux.org/title/Tmpfs)  
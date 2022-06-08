---
title: 关于SSD写入放大
date: 2022-04-14 18:46:12
tags:
- SSD
- Write-Amplification
- LSM-Tree
abbrlink: 'write-amplification'
categories:
- 笔记
---
纪念上一块编译报废的固态。
<!-- more -->

## 为什么会有写入放大

### gc造成的写入放大

SSD的写入是先擦除再写入（Erase-Before-Write）

## LSM Tree

李世民树，是李世民体恤SSD读写太高耗费国库所种的树 (大雾

LSM Tree用的时候一般把日志等文件设计成在磁盘上顺序读写，这样些性能会比较好

## 说一下Btrfs写入放大的问题

我之前看到Btrfs在放数据库的子卷和

关于Btrfs文件系统数据库要关闭COW的问题，直接贴一下fc老师的话

>虛擬機鏡像，InnoDB 那種底層是 b-tree 結構的數據庫（leveldb/rocksdb/myrocks 那種底層是 lsm tree 的不需要
>因為這些 b-tree 數據庫自己做 4k/8k/16k 規模的 b-tree node 隨機讀寫，隨機讀寫和 cow fs 相性不好，尤其開了 compress 的 btrfs 每個 compressed extent 可以到 128K 就會有 32 倍的寫放大。（順便這種讀寫方式和有 ftl 的 ssd ，有 stl 的 smr 的相性也不好…

## 利用内存减少SSD读写

众所周知，SSD的读写寿命比较低，家用的SSD写入比较多可能一两年就报废了；而内存在你电脑的其他零件都报废之前几乎不存在读写寿命的问题，大概率是最后一个坏的。可是MOS-FET当年上课的老师说这个东西没那么容易坏的，是从MLC、TLC、QLC一代不如一代， 像傲腾之类的SSD也不太容易写坏

如果内存够大，扔到内存里

## 链接

[Ask HN: Which Linux filesystem “produce” less wear and tear on SSD NAND?](https://news.ycombinator.com/item?id=22767106)  
[Выявляем процессы с дисковой активностью в Linux](https://habr.com/ru/post/476414/)  一篇Btrfs Write Amplification讨论  

[The Log-Structured Merge-Tree (LSM-Tree)](https://www.cs.umb.edu/~poneil/lsmtree.pdf)  
[A Comparison of Fractal Trees to Log-Structured Merge (LSM) Trees](http://www.pandademo.com/wp-content/uploads/2017/12/A-Comparison-of-Fractal-Trees-to-Log-Structured-Merge-LSM-Trees.pdf)   
[Write amplification analysis in flash-based solid state drives](https://dl.acm.org/doi/10.1145/1534530.1534544)   
[Thread: Leap & TW: Enormously high write amplification with BTRFS](https://forums.opensuse.org/showthread.php/548476-Leap-amp-TW-Enormously-high-write-amplification-with-BTRFS)   


[ArchWiki Solid state drive](https://wiki.archlinux.org/title/Solid_state_drive)   
[ArchWiki Tmpfs](https://wiki.archlinux.org/title/Tmpfs)  

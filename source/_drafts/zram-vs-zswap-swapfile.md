---
title: 还是用上了zram
tags:
- zram
abbrlink: 'zram-vs-zswap-swapfile'
---
之前对于zram和zswap的一点迷惑。（当然如果你内存足够大，这些你都不需要）
<!-- more -->

以前以为zram适合小内存的设备（比如手机上这种），后来发现这个东西对CPU的性能要求不低的。小内存的情况+CPU弱鸡，频繁压缩很容易造成卡顿

## 链接

https://www.kernel.org/doc/html/latest/admin-guide/blockdev/zram.html#writeback

https://lwn.net/Articles/545244/
http://www.wowotech.net/memory_management/zram.html


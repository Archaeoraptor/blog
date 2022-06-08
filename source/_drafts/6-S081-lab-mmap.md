---
title: 6.S081 Lab mmap 笔记
date: 2022-04-15 19:17:29
tags:
- 6.S081
- mmap
abbrlink: '6-S081-lab-mmap'
categories:
- Linux&Unix
---
mmap实现细节坑略多。本来想再写点linux的sendfile和splice相关的内容的，但是源码比较麻烦，算了，先大概聊聊原理吧
<!-- more -->

如果不在乎性能，那么文件的读写只要read和write这样的syscall就可以了，mmap、sendfile、slice等技术都是为了性能，通过某些方式减少系统调用和拷贝。

read和write作为系统调用是很慢的，大概要经过磁盘数据->内核page cache

### mmap

mmap是很经典的技术，思路也很简单。申请内存的时候经常用mmap。思路也比较简单，将文件映射到内存里，然后像读写文件一样读写内存。mmap将文件映射到了进程的虚拟地址空间（vma），操作内存可以达到直接操作块设备的效果。（注意这里和直接将文件放在物理内存里是不一样的, mmap可以直接映射一个很大的文件，超过物理内存，当用到的时候才会加载进物理内存）

```c
#include <sys/mman.h>

void *mmap(void *addr, size_t length, int prot, int flags,
                  int fd, off_t offset);
int munmap(void *addr, size_t length);
```

mmap有三种，MAP_SHARED，MAP_PRIVATE和MAP_SHAEED_VALIDATE，MAP_SHARE是共享（修改对其他进程可见）， MAP_PRIVATE是不共享（creating a private cow write mapping， 其他进程不可见）, MAP_SHAEED_VALIDATE是linux在

既然映射到了内存里，我们一样可以在`/proc`里面找到：

```c
cat /proc/1/maps
```

我之前有一个误解是mmap的开销相比read、write小很多，这多好啊，都用mmap呗。后来发现mmap很容易出现各种脏页和不一致的问题。如果文件需要频繁更新，那落盘的时候也需要更多的

### sendfile

### splice

splice通过管道在两个socket之间传数据

## Lab的准备工作：xv6中page fault相关代码

我们来看一下xv6的 page fault 和 vma 等相关内容

```c
// Remove npages of mappings starting from va. va must be
// page-aligned. The mappings must exist.
// Optionally free the physical memory.
void uvmunmap(pagetable_t pagetable, uint64 va, uint64 npages, int do_free) {
    uint64 a;
    pte_t *pte;

    if ((va % PGSIZE) != 0)
        panic("uvmunmap: not aligned");

    for (a = va; a < va + npages * PGSIZE; a += PGSIZE) {
        if ((pte = walk(pagetable, a, 0)) == 0)
            panic("uvmunmap: walk");
        if ((*pte & PTE_V) == 0)
            panic("uvmunmap: not mapped");
        if (PTE_FLAGS(*pte) == PTE_V)
            panic("uvmunmap: not a leaf");
        if (do_free) {
            uint64 pa = PTE2PA(*pte);
            kfree((void *)pa);
        }
        *pte = 0;
    }
}
```

## Lab

>You should implement enough mmap and munmap functionality to make the mmaptest test program work. If mmaptest doesn't use a mmap feature, you don't need to implement that feature.

这个lab的意思很明确，就是让我们实现一个naive的mmap。这里Lab的测试要求不是很高，mmap在MAP_SHARED的时候不共享物理页也是可以的。

首先我们抄一个VMA

## 番外：mmap用在数据库上是否是个好主意

CS15-445那个课上说了好多次mmap不适合数据库

https://db.cs.cmu.edu/mmap-cidr2022/  

[Are You Sure You Want to Use MMAP in Your
Database Management System?](https://db.cs.cmu.edu/papers/2022/cidr2022-p13-crotty.pdf)

RavenDB CEO的回应：[re: Are You Sure You Want to Use MMAP in Your Database Management System?](https://ravendb.net/articles/re-are-you-sure-you-want-to-use-mmap-in-your-database-management-system)  

## 链接

<https://man7.org/linux/man-pages/man2/mmap.2.html>

<https://man7.org/linux/man-pages/man3/mmap.3p.html>

[mmap详解](https://nieyong.github.io/wiki_cpu/mmap详解.html)

下面是io_uring相关的东西

[[译] Linux 异步 I/O 框架 io_uring：基本原理、程序示例与性能压测（2020）](https://arthurchiao.art/blog/intro-to-io-uring-zh/)  
[AIO 的新归宿：io_uring](https://zhuanlan.zhihu.com/p/62682475)  


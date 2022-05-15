---
title: 6.S081 Lab xv6 lazy page allocation（补档）
tags:
- 6.S081
- sbrk
categories:
- Linux&Unix
date: 2022-04-15 19:17:29
abbrlink: '6-S081-lab-lazy-page-allocation'
---
最近在看TLPI, 看到brk和sbrk的时候突然发现6.S081居然还有个和sbrk相关的lab。再一看我竟然还没有做？我怎么记得我明明做完了的，再一看是2020 fall的Lab内容，2021年的没有，顺手做了吧。
<!-- more -->

## 关于brk和sbrk

```c
#include <unistd.h>

int brk(void *addr);
void *sbrk(intptr_t increment);
```

假设堆是向上生长的，堆顶的地址就是 program break。brk和sbrk用来改变program break（就是堆的边界），brk是直接改变地址，sbrk是一个increment的偏移量，当increment>0时向上移动program break，作用相当于增加堆的大小，increment小于0减少堆的大小。

brk一般用的比较少，大多数时候申请内存用malloc。malloc分配小块内存的时候就是调用brk/sbrk实现的，当可用内存不够的时候调用brk扩容。

## Lab

### Eliminate allocation from sbrk() (easy)

>Your first task is to delete page allocation from the sbrk(n) system call implementation, which is the function sys_sbrk() in sysproc.c. The sbrk(n) system call grows the process's memory size by n bytes, and then returns the start of the newly allocated region (i.e., the old size). Your new sbrk(n) should just increment the process's size (myproc()->sz) by n and return the old size. It should not allocate memory -- so you should delete the call to growproc() (but you still need to increase the process's size!).

这里我们先把sbrk的分配内存删了。sbrk的分配内存操作调用growproc函数完成，growproc在n>的时候分配内存，在n<0的时候调用uvdealloc

```c
// Grow or shrink user memory by n bytes.
// Return 0 on success, -1 on failure.
int growproc(int n) {
    uint sz;
    struct proc *p = myproc();

    sz = p->sz;
    if (n > 0) {
        if ((sz = uvmalloc(p->pagetable, sz, sz + n)) == 0) {   // 分配内存（grow）
            return -1;
        }
    } else if (n < 0) {
        sz = uvmdealloc(p->pagetable, sz, sz + n);  // 收缩内存（shrink）
    }
    p->sz = sz;
    return 0;
}
```

growproc调用uvmalloc和uvdealloc进行内存的扩容和收缩

```c
// Allocate PTEs and physical memory to grow process from oldsz to
// newsz, which need not be page aligned.  Returns new size or 0 on error.
uint64 uvmalloc(pagetable_t pagetable, uint64 oldsz, uint64 newsz) {
    char *mem;
    uint64 a;

    if (newsz < oldsz)
        return oldsz;

    oldsz = PGROUNDUP(oldsz);
    for (a = oldsz; a < newsz; a += PGSIZE) {
        mem = kalloc();
        if (mem == 0) {
            uvmdealloc(pagetable, a, oldsz);
            return 0;
        }
        memset(mem, 0, PGSIZE);
        if (mappages(pagetable, a, PGSIZE, (uint64)mem,
                     PTE_W | PTE_X | PTE_R | PTE_U) != 0) {
            kfree(mem);
            uvmdealloc(pagetable, a, oldsz);
            return 0;
        }
    }
    return newsz;
}
```

uvdealloc调用了kalloc分配内存, xv6的kalloc实现也非常的简单，加一个自旋锁然后把要分配的内存区域都用memset置为5（这个比linux的kalloc简单太多了，xv6作为一个教学系统太棒了，基本不需要注释，一看就懂了，不会陷入到各种复杂的实现和细节里面）

```c
// Allocate one 4096-byte page of physical memory.
// Returns a pointer that the kernel can use.
// Returns 0 if the memory cannot be allocated.
void *kalloc(void) {
    struct run *r;

    acquire(&kmem.lock);
    r = kmem.freelist;
    if (r)
        kmem.freelist = r->next;
    release(&kmem.lock);

    if (r)
        memset((char *)r, 5, PGSIZE); // fill with junk
    return (void *)r;
}
```

memset的实现也相当简单，遍历然后复制

```c
void *memset(void *dst, int c, uint n) {
    char *cdst = (char *)dst;
    int i;
    for (i = 0; i < n; i++) {
        cdst[i] = c;
    }
    return dst;
}
```

然后我们继续看 sys_sbrk 中的growproc，argint用来获取第0号系统调用的参数，放在n中（n是一个int指针）, 运行结束后n是sbrk的偏移大小

```c
// Fetch the nth 32-bit system call argument.
int argint(int n, int *ip) {
    *ip = argraw(n);
    return 0;
}
```

所以我们只要注释掉sys_sbrk中的growproc就可以了，只让它对myproc->sz+n

```c
// Grow or shrink user memory by n bytes.
// Return 0 on success, -1 on failure.
uint64 sys_sbrk(void) {
    int addr;
    int n;

    if (argint(0, &n) < 0)
        return -1;
    addr = myproc()->sz;
    myproc() -> sz = myproc() -> sz + n;
    // if (growproc(n) < 0)
    //     return -1;
    return addr;
}
```

这里myproc是一个proc结构体，sz是进程内存的大小（Size of process memory (bytes)）。.

删掉之后当然会 page fault 了，在接下来的Lab中我们去处理这个 page fault 然后实现 lazy allocation

```bash
$ echo hi
usertrap(): unexpected scause 0x000000000000000f pid=3
            sepc=0x00000000000012ac stval=0x0000000000004008
panic: uvmunmap: not mapped
```

scause寄存器的值是0x000000000000000f（10进制是15，查手册可知这是一个store page fault），进程的pid是3。
stval的值是page fault时虚拟内存的地址。

### Lazy allocation (moderate)

>Modify the code in trap.c to respond to a page fault from user space by mapping a newly-allocated page of physical memory at the faulting address, and then returning back to user space to let the process continue executing. You should add your code just before the printf call that produced the "usertrap(): ..." message. Modify whatever other xv6 kernel code you need to i到n order to get echo hi to work.

然后我们实现懒分配。只有当需要申请实际的物理地址的时候，触发page fault，然后才会进行实际的分配。首先我们修改usertrap的行为，检测未分配的内存引起的page fault时进行分配。
page fault的原因存在SCAUSE寄存器中，我们根据r_scause()等于13或者15来判断异常原因，用r_stval取出STVAL寄存器的值进行判断

>stval保存了陷入(trap)的附加信息:地址例外中出错
的地址、发生非法指令例外的指令本身,对于其他异常,它的值为 0

```c
// Supervisor Trap Value
static inline uint64 r_stval() {
    uint64 x;
    asm volatile("csrr %0, stval" : "=r"(x));
    return x;
}
```

直接调用r_stval就可以了，我之前还傻乎乎的自己内联汇编。。。我们取出STVAL寄存器的值就是出错的virtual address

```c
    } else if (r_scause() == 15 || r_scause() == 13) {
        uint64 va = r_stval();
        printf("page fault %p\n", va);
        uint64 ka = (uint64)kalloc();
        if (ka == 0) { // 内存分配失败，直接kill
            p->killed = 1;
        } else {
            memset((void *)ka, 0, PGSIZE);
            va = PGROUNDDOWN(va);
            if (mappages(p->pagetable, va, PGSIZE, ka, PTE_W | PTE_U | PTE_R) !=
                0) {
                kfree((void *)ka);
                p->killed = 1;
            }
        }
    } else if ((which_dev = devintr()) != 0) {
```

其中`#define PGROUNDDOWN(a) (((a)) & ~(PGSIZE - 1))`，是用来4k地址对齐的（这里PGSIZE是4k，如果PGSIZE改了，比如大页，就是其他大小的地址对齐），PGROUNDUP(sz)向上对齐，PGROUNDDOWN(a)向下对齐， 比如a是114或者514，PGROUNDDOWN(a)会向下对齐到0

 `manpages`用来给virtual address 创建PTE

```c
// Create PTEs for virtual addresses starting at va that refer to
// physical addresses starting at pa. va and size might not
// be page-aligned. Returns 0 on success, -1 if walk() couldn't
// allocate a needed page-table page.
int mappages(pagetable_t pagetable, uint64 va, uint64 size, uint64 pa,
             int perm) {
    uint64 a, last;
    pte_t *pte;

    a = PGROUNDDOWN(va);
    last = PGROUNDDOWN(va + size - 1);
    for (;;) {
        if ((pte = walk(pagetable, a, 1)) == 0)
            return -1;
        if (*pte & PTE_V)
            panic("remap");
        *pte = PA2PTE(pa) | perm | PTE_V;
        if (a == last)
            break;
        a += PGSIZE;
        pa += PGSIZE;
    }
    return 0;
}
```

现在再次`echo hi`已经不会page fault了

```bash
$ echo hi
page fault 0x0000000000004008
page fault 0x0000000000013f48
panic: uvmunmap: not mapped
```

然后我们去map它，修改`uvmunmap`，直接改成

### Lazytests and Usertests (moderate)

这个Lab的usertests很耗时间，我这里大概跑一次要五分钟左右。

我们要去处理一下sbrk的边界条件，如果sbrk的参数<0, 我们要shrink，这个简单， 直接诶调用uvmdealloc或者growproc就可以了。如果进程访问了比sbrk还要高的地址，直接panic， 然后当address合法但没有分配的时候我们去分配。

`sys_sbrk`加上

```c
    if (n < 0) {
        uvmdealloc(myproc()->pagetable, addr, myproc()->sz);
    }
```

然后是父子进程的地址问题，相关的函数在`uvmcopy`， 我们同样注释掉那两个panic

```c
// Given a parent process's page table, copy
// its memory into a child's page table.
// Copies both the page table and the
// physical memory.
// returns 0 on success, -1 on failure.
// frees any allocated pages on failure.
int uvmcopy(pagetable_t old, pagetable_t new, uint64 sz) {
    pte_t *pte;
    uint64 pa, i;
    uint flags;
    char *mem;

    for (i = 0; i < sz; i += PGSIZE) {
        if ((pte = walk(old, i, 0)) == 0)
            // panic("uvmcopy: pte should exist");
            continue;
        if ((*pte & PTE_V) == 0)
            // panic("uvmcopy: page not present");
            continue;
        pa = PTE2PA(*pte);
        flags = PTE_FLAGS(*pte);
        if ((mem = kalloc()) == 0)
            goto err;
        memmove(mem, (char *)pa, PGSIZE);
        if (mappages(new, i, PGSIZE, (uint64)mem, flags) != 0) {
            kfree(mem);
            goto err;
        }
    }
    return 0;

err:
    uvmunmap(new, 0, i / PGSIZE, 1);
    return -1;
}
```

>Handle the case in which a process passes a valid address from sbrk() to a system call such as read or write, but the memory for that address has not yet been allocated.

我一开始以为要去改read和write，然后对read和write这两个syscall动了手脚跑过了测试（就像当初syscall Lab那样），再一想不对，怎么能改syscall呢。应该调整`vm.c`中的行为，这样对于每一个syscall都是有效的（system call such as read or write）

## 链接

<https://man7.org/linux/man-pages/man2/sbrk.2.html>  
<https://pdos.csail.mit.edu/6.S081/2020/labs/lazy.html>  

---
title: 6.S081 Lab Lock 笔记
date: 2022-04-15 17:07:01
tags:
- 6.S081
- LRU
abbrlink: '6-S081-lab-lock'
---
一个比较有意思的Lab
<!-- more -->

## xv6中的锁

### spinlock

一个naive的spinlock实现比较简单（我们不考虑中断），直接看xv6的实现吧

```c
// Mutual exclusion lock.
struct spinlock {
  uint locked;       // Is the lock held? 1表示locked，0表示unlocked

  // For debugging:
  char *name;        // Name of lock.
  struct cpu *cpu;   // The cpu holding the lock.
};
```

初始化

```c
void
initlock(struct spinlock *lk, char *name)
{
  lk->name = name;
  lk->locked = 0;
  lk->cpu = 0;
}
```

`acquire`获取锁， test_and_set 是一个原子操作（在RISC-V中是amoswap指令，对于多个CPU也是原子的）
注意`__sync_synchronize();`是为了加上内存屏障强制保证顺序执行的。

```c
// Acquire the lock.
// Loops (spins) until the lock is acquired.
void
acquire(struct spinlock *lk)
{
  push_off(); // disable interrupts to avoid deadlock.  关闭中断
  if(holding(lk))   // 检查是否已经持有锁（防止重复上锁）
    panic("acquire");   //如果持有锁，panic

  // On RISC-V, sync_lock_test_and_set turns into an atomic swap:
  //   a5 = 1
  //   s1 = &lk->locked
  //   amoswap.w.aq a5, a5, (s1)
  while(__sync_lock_test_and_set(&lk->locked, 1) != 0)  // 等于1表示锁被别人持有，这里一直等到没人持有spinlock
    ;

  // Tell the C compiler and the processor to not move loads or stores
  // past this point, to ensure that the critical section's memory
  // references happen strictly after the lock is acquired.
  // On RISC-V, this emits a fence instruction.
  __sync_synchronize(); //强制顺序执行

  // Record info about lock acquisition for holding() and debugging.
  lk->cpu = mycpu();
}
```

注意spinlock持有的是有。在acquire之前要关闭中断，如果在acquire的过程中有中断，会导致死锁。比如当前进程p1已经acquire了spinlock，然后一个中断（interrupt）到来，进程p2又试图获取spinlock；但是此次spinlock被p1持有，然后就死锁了（一直在`while(__sync_lock_test_and_set(&lk->locked, 1) != 0)`出不去了）

释放锁也和acquire差不多

```c
// Release the lock.
void
release(struct spinlock *lk)
{
  if(!holding(lk))
    panic("release");

  lk->cpu = 0;

  // Tell the C compiler and the CPU to not move loads or stores
  // past this point, to ensure that all the stores in the critical
  // section are visible to other CPUs before the lock is released,
  // and that loads in the critical section occur strictly before
  // the lock is released.
  // On RISC-V, this emits a fence instruction.
  __sync_synchronize(); //强制顺序执行

  // Release the lock, equivalent to lk->locked = 0.
  // This code doesn't use a C assignment, since the C standard
  // implies that an assignment might be implemented with
  // multiple store instructions.
  // On RISC-V, sync_lock_release turns into an atomic swap:
  //   s1 = &lk->locked
  //   amoswap.w zero, zero, (s1)
  __sync_lock_release(&lk->locked); //这里也是原子操作

  pop_off();    //打开中断
}
```

检查持有，这个比较简单不多说了

```c
// Check whether this cpu is holding the lock.
// Interrupts must be off.
int
holding(struct spinlock *lk)
{
  int r;
  r = (lk->locked && lk->cpu == mycpu());
  return r;
}
```

## Lab


## 链接


[如何设计真正高性能的 spin_lock？ - 知乎](https://www.zhihu.com/question/55764216)  
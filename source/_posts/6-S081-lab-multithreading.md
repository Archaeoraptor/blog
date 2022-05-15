---
title: 6.S081 Lab Multithreading 笔记
date: 2022-04-20 19:27:58
tags:
- 6.S081
- thread
abbrlink: '6-S081-lab-multithreading'
categories:
- Linux&Unix
---
实现一个用户态线程uthread，就照着xv6的内核态线程抄呗。怎么什么FUSE啊、UIO啊，什么东西都想往用户态搬啊，连RCU都有人想往用户态搬。
<!-- more -->

## xv6中的process

### process的状态

xv6的线程状态包括 UNUSED, USED, SLEEPING, RUNNABLE, RUNNING, ZOMBIE

![](6-S081-lab-multithreading/1650459636.png)

### process调度

用户态进程的切换是需要内核态的 kstack scheduler的

![](6-S081-lab-multithreading/1650459961.png)

我们下面来看调度过程，相比linux里面的调度器这个简单多了。就是一个最简单的时间片轮转。

首先我们看proc这个结构体（就相当于linux里面的task这个结构体）

```c
// Per-process state
struct proc {
  struct spinlock lock;

  // p->lock must be held when using these:
  enum procstate state;        // Process state
  void *chan;                  // If non-zero, sleeping on chan
  int killed;                  // If non-zero, have been killed
  int xstate;                  // Exit status to be returned to parent's wait
  int pid;                     // Process ID

  // wait_lock must be held when using this:
  struct proc *parent;         // Parent process

  // these are private to the process, so p->lock need not be held.
  uint64 kstack;               // Virtual address of kernel stack
  uint64 sz;                   // Size of process memory (bytes)
  pagetable_t pagetable;       // User page table
  struct trapframe *trapframe; // data page for trampoline.S
  struct context context;      // swtch() here to run process
  struct file *ofile[NOFILE];  // Open files
  struct inode *cwd;           // Current directory
  char name[16];               // Process name (debugging)
};
```

在process切换的时候，保存proc的trapframe和context用来恢复，然后还有proc的状态（STATE）、pid等。

在切换的时候主要的工作是context和trapframe的保存和恢复。context被翻译成上下文，然后当年书里说半天我也没搞懂这到底是个啥玩意。后来看了xv6的源码才知道这是个啥。到底是谁最先把context翻译成上下文的啊？？？？ 我们直接看源码， `kernel/proc.h`

```c
// Saved registers for kernel context switches.
struct context {
  uint64 ra; # return address, 返回地址
  uint64 sp; # stack pointer，栈指针寄存器

  // callee-saved
  uint64 s0;
  uint64 s1;
  uint64 s2;
  uint64 s3;
  uint64 s4;
  uint64 s5;
  uint64 s6;
  uint64 s7;
  uint64 s8;
  uint64 s9;
  uint64 s10;
  uint64 s11;
};
```

context是一个结构体，里面是寄存器的值（用来保存和恢复process的状态）

trapframe也是一个结构体，放的东西更多了点。比如process的内核页表、比如kernel_sp寄存器，比如epc寄存器

```c
struct trapframe {
  /*   0 */ uint64 kernel_satp;   // kernel page table
  /*   8 */ uint64 kernel_sp;     // top of process's kernel stack
  /*  16 */ uint64 kernel_trap;   // usertrap()
  /*  24 */ uint64 epc;           // saved user program counter
  /*  32 */ uint64 kernel_hartid; // saved kernel tp
  /*  40 */ uint64 ra;
  /*  48 */ uint64 sp;
  /*  56 */ uint64 gp;
  /*  64 */ uint64 tp;
  /*  72 */ uint64 t0;
  /*  80 */ uint64 t1;
  /*  88 */ uint64 t2;
  /*  96 */ uint64 s0;
  /* 104 */ uint64 s1;
  /* 112 */ uint64 a0;
  /* 120 */ uint64 a1;
  /* 128 */ uint64 a2;
  /* 136 */ uint64 a3;
  /* 144 */ uint64 a4;
  /* 152 */ uint64 a5;
  /* 160 */ uint64 a6;
  /* 168 */ uint64 a7;
  /* 176 */ uint64 s2;
  /* 184 */ uint64 s3;
  /* 192 */ uint64 s4;
  /* 200 */ uint64 s5;
  /* 208 */ uint64 s6;
  /* 216 */ uint64 s7;
  /* 224 */ uint64 s8;
  /* 232 */ uint64 s9;
  /* 240 */ uint64 s10;
  /* 248 */ uint64 s11;
  /* 256 */ uint64 t3;
  /* 264 */ uint64 t4;
  /* 272 */ uint64 t5;
  /* 280 */ uint64 t6;
};
```

说一下trapframe和context的区别，context保存的是线程切换（比如调度器切换线程）时候的数据（寄存器），trapframe保存的是trap的时候（切换内核态时候）的数据

#### 运行状态切换

进入RUNNING是由`scheduler`处理的，这里就直接是一个简单的for循环，加锁找到可用的直接将状态变成RUNNING释放锁。

```c
void
scheduler(void)
{
  struct proc *p;
  struct cpu *c = mycpu();
  
  c->proc = 0;
  for(;;){
    // Avoid deadlock by ensuring that devices can interrupt.
    intr_on();

    for(p = proc; p < &proc[NPROC]; p++) {
      acquire(&p->lock);
      if(p->state == RUNNABLE) {
        // Switch to chosen process.  It is the process's job
        // to release its lock and then reacquire it
        // before jumping back to us.
        p->state = RUNNING;
        c->proc = p;
        swtch(&c->context, &p->context);    // 保存context（上下文）

        // Process is done running for now.
        // It should have changed its p->state before coming back.
        c->proc = 0;
      }
      release(&p->lock);
    }
  }
}
```

这里swtch是保存并切换context。`swtch(&c->context, &p->context);`表示保存c的context, 加载p的context。我们直接看一下`swtch.S`的代码就明白了：

```armasm
.globl swtch
swtch:
        sd ra, 0(a0)
        sd sp, 8(a0)
        sd s0, 16(a0)
        .......

        ld ra, 0(a1)
        ld sp, 8(a1)
        ld s0, 16(a1)
        .......

        ret
```

退出的过程是调用了`yield`，从RUNNING进入RUNNABLE

```c
// Give up the CPU for one scheduling round.
void
yield(void)
{
  struct proc *p = myproc();
  acquire(&p->lock);    // 这里也要加一个自旋锁，如果不加，在yield过程中来一个中断之类的会出问题
  p->state = RUNNABLE;
  sched();  // 这里调用sched切换进程（恢复上下文等操作）
  release(&p->lock);
}
```

yield 调用了 sched

```c
void
sched(void)
{
  int intena;
  struct proc *p = myproc();

  if(!holding(&p->lock))    // 必须持有锁
    panic("sched p->lock");
  if(mycpu()->noff != 1)    //中断是关的
    panic("sched locks");
  if(p->state == RUNNING)   //之前已经设为RUNNABLE，肯定不能是RUNNING
    panic("sched running");
  if(intr_get())    // 中断不能是enabled
    panic("sched interruptible");

  intena = mycpu()->intena; //将itena置为当前CPU的itena
  swtch(&p->context, &mycpu()->context);    //恢复context（上下文）
  mycpu()->intena = intena;
}
```

说一下这个itena，定义在`kernel/proc.h`中，就是中断（interrupt）是否在push_off之前enable了

```c
// Per-CPU state.
struct cpu {
  struct proc *proc;          // The process running on this cpu, or null.
  struct context context;     // swtch() here to enter scheduler().
  int noff;                   // Depth of push_off() nesting.
  int intena;                 // Were interrupts enabled before push_off()?
};
```

从RUNNING退出到别的状态可能是收到了中断，也可能是进入SLEEPING（比如耗时较长的文件IO），也可能是被kill了或者父进程被kill了进入了ZOMBIE状态

```c
int
kill(int pid)
{
  struct proc *p;
 
  for(p = proc; p < &proc[NPROC]; p++){
    acquire(&p->lock);
    if(p->pid == pid){
      p->killed = 1;
      if(p->state == SLEEPING){
        // Wake process from sleep().
        p->state = RUNNABLE;
      }
      release(&p->lock);
      return 0;
    }
    release(&p->lock);
  }
  return -1;
}
```

#### 睡眠和唤醒

wakeup比较简单，就正常的加锁然后置为RUNNABLE就好了

```c
// Wake up all processes sleeping on chan.
// Must be called without any p->lock.
void
wakeup(void *chan)
{
  struct proc *p;

  for(p = proc; p < &proc[NPROC]; p++) {
    if(p != myproc()){
      acquire(&p->lock);
      if(p->state == SLEEPING && p->chan == chan) {
        p->state = RUNNABLE;
      }
      release(&p->lock);
    }
  }
}
```

sleep这里要注意！这个spinlock是为了保证在执行sleep的时候没有别的进程调用wakeup。开始执行sleep之后lk就可以释放掉了。

```c
// Atomically release lock and sleep on chan.
// Reacquires lock when awakened.
void
sleep(void *chan, struct spinlock *lk)
{
  struct proc *p = myproc();
  
  // Must acquire p->lock in order to
  // change p->state and then call sched.
  // Once we hold p->lock, we can be
  // guaranteed that we won't miss any wakeup
  // (wakeup locks p->lock),
  // so it's okay to release lk.

  acquire(&p->lock);  //DOC: sleeplock1
  release(lk);

  // Go to sleep.
  p->chan = chan;
  p->state = SLEEPING;

  sched();

  // Tidy up.
  p->chan = 0;

  // Reacquire original lock.
  release(&p->lock);
  acquire(lk);
}
```

## Lab

这个Lab有点意思，实现可以直接照抄内核态线程，做一个naive的用户态线程

### Uthread: switching between threads (moderate)

用户态线程的状态只有三种，FREE, RUNNING, RUNNABLE。状态机比较简单。`thread_init`初始化的时候状态设为RUNNING

创建a,b,c三个用户态thread,然后每个线程打印一次后就执行`thread_yield`切换线程，并将当前线程的状态设置为RUNNABLE

```c
void thread_yield(void) {
    current_thread->state = RUNNABLE;
    thread_schedule();
}
```

然后在线程结束的时候将状态置为FREE, `current_thread->state = FREE;`，调用

`thread_create`传进来的参数是一个函数指针（比如thread_a）, 我们把函数指针存入ra寄存器，用来做恢复。`thread_create`实现可以参考`kernel/proc.c`里面的`allcproc()`

```c
void thread_create(void (*func)()) {
    struct thread *t;

    for (t = all_thread; t < all_thread + MAX_THREAD; t++) {
        if (t->state == FREE)
            break;
    }
    t->state = RUNNABLE;
    t->thread_context.ra = (uint64)func;
    t->thread_context.sp = t->stack + STACK_SIZE;
}
```

我们把`kernel/proc.h`中的context抄过来

```c
// Saved registers for user context switches.
struct user_context {
  uint64 ra;
  uint64 sp;

  // callee-saved
  uint64 s0;
  uint64 s1;
  uint64 s2;
  uint64 s3;
  uint64 s4;
  uint64 s5;
  uint64 s6;
  uint64 s7;
  uint64 s8;
  uint64 s9;
  uint64 s10;
  uint64 s11;
};

struct thread {
    char stack[STACK_SIZE]; /* the thread's stack */
    int state;              /* FREE, RUNNING, RUNNABLE */
    struct user_context thread_context;
};
```

然后`user/uthread_switch.S`中的`thread_switch`照抄`kernel/swtch.S`实现状态切换，不过`thread_switch`这里的上下文只保存Callee-saved的寄存器就可以了，所以我们删掉`sd ra, 0(a0)`和`ld ra, 0(a1)`

**sp寄存器是callee寄存器，只有ra寄存器是caller寄存器**，这个不能删，删了之后用户线程没法切换了！一开始记错了把sp寄存器也记成caller寄存器了，没输出调了好久

```armasm
# void thread_switch(struct context *old, struct context *new);
thread_switch:
    sd sp, 8(a0)
    sd s0, 16(a0)
    sd s1, 24(a0)
    sd s2, 32(a0)
    sd s3, 40(a0)
    sd s4, 48(a0)
    sd s5, 56(a0)
    sd s6, 64(a0)
    sd s7, 72(a0)
    sd s8, 80(a0)
    sd s9, 88(a0)
    sd s10, 96(a0)
    sd s11, 104(a0)

    ld sp, 8(a1)
    ld s0, 16(a1)
    ld s1, 24(a1)
    ld s2, 32(a1)
    ld s3, 40(a1)
    ld s4, 48(a1)
    ld s5, 56(a1)
    ld s6, 64(a1)
    ld s7, 72(a1)
    ld s8, 80(a1)
    ld s9, 88(a1)
    ld s10, 96(a1)
    ld s11, 104(a1)

	ret    /* return to ra */
```

然后在`thread_switch`里面直接调用`thread_switch((uint64)t, (uint64)current_thread);`

然后。。。报错了。调了一下午没找到哪出问题了，gdb打出来的调试信息看不出来。这个报错跟当初页表那个很像，debug的时候被误导了。

```bash
$ uthread
usertrap(): unexpected scause 0x000000000000000f pid=3
            sepc=0x0000000000000002 stval=0xfffffffffffffff8
```

调了半天发现是我thread_switch的函数参数传错了，传进去的应该是context，寄存器a0是第一个参数，寄存器a1是参数，放的是寄存器的struct，然后按照地址偏移分别对应s0，s1等Callee寄存器的值

```c
thread_switch((uint64)&t->thread_context, (uint64)&current_thread->thread_context);
```

终于正常了

```c
$ uthread
310514718thread_a started
thread_b started
thread_c started
thread_c 0
thread_a 0
thread_b 0
thread_c 1
thread_a 1
thread_b 1
thread_c 2
thread_a 2
......
thread_c 99
thread_a 99
thread_b 99
thread_c: exit after 100
thread_a: exit after 100
thread_b: exit after 100
thread_schedule: no runnable threads
```

### Using threads (moderate)

这个就是学一下怎么用pthread（这个是在unix/linux里面，不是在qemu里），练习一下加mutex锁。`ph.c`里面有一个并发不安全的哈希表，就是一个简单的取模，然后放到相应的bucket里面，bucket中有重复的key就依次加到链表的后面。实现就和leetcode 706 设计哈希映射的官方题解差不多。这里bucket数量是5，所以很容易发生冲突，缓解hash冲突的一般做法是bucket去一个更大的质数，或者设计其他更好的hash函数减少hash冲突，或者我们不要链表，用开放定址或者rehash去解决hash冲突

```bash
$make ph
$./ph 1
100000 puts, 5.690 seconds, 17573 puts/second
0: 0 keys missing
100000 gets, 5.491 seconds, 18211 gets/second
```

我们把线程开多一点

```bash
$./ph 10
100000 puts, 0.333 seconds, 300217 puts/second
2: 63594 keys missing
7: 63594 keys missing
1: 63594 keys missing
9: 63594 keys missing
5: 63594 keys missing
8: 63594 keys missing
6: 63594 keys missing
4: 63594 keys missing
0: 63594 keys missing
3: 63594 keys missing
1000000 gets, 5.871 seconds, 170320 gets/second
```

然后我们把线程开到100，现在打字都卡了, 成千上万个 key missng。这里的hashmap在并发写的时候会冲突，就像go的map一样并发写的时候会直接报`fatal error: concurrent map writes`。读是没有关系的，但是`ph.c`第26行那个insert在写的时候是向entry（存key和value的链表后面加），并发写会把同时正在插入的覆盖掉。

Go map in actions 中有一个非常简单的给map加锁的例子

```go
var counter = struct{
    sync.RWMutex
    m map[string]int
}{m: make(map[string]int)}

counter.RLock()
n := counter.m["some_key"]
counter.RUnlock()
fmt.Println("some_key:", n)

counter.Lock()
counter.m["some_key"]++
counter.Unlock()
```

我们仿照go的map写入加`sync.Mutex`，给insert加互斥锁。

```c
pthread_mutex_t lock; // declare a lock

static void *put_thread(void *xa) {
    int n = (int)(long)xa; // thread number
    int b = NKEYS / nthread;

    for (int i = 0; i < b; i++) {
        pthread_mutex_lock(&lock);
        put(keys[b * n + i], n);
        pthread_mutex_unlock(&lock);
    }

    return NULL;
}
```

然后就没有missing了

```bash
$make ph
$./ph 2
100000 puts, 5.522 seconds, 18111 puts/second
1: 0 keys missing
0: 0 keys missing
200000 gets, 5.628 seconds, 35535 gets/second
```

但是这样有一个问题，速度没什么优势，把nthread开到10，puts用时6.094 seconds，还不如单线程。`make grade`的时候只能跑过`ph_safe`，没法通过`ph_fast`的。

想要速度我们需要更细粒度的锁，我们仿照go的concurrent-map那样分片加锁的方案在insert插入的时候给对应的bucket加锁。（go的sync.Map的那个加锁比较适合读多写少的方案，这里大量写入优势不大）这里我们只要对每个bucket单独加锁就可以了，问题转换为对entry链表加锁，我们用NBUCKET把小锁分别对NBUCKET个entry链表加锁。

```c
pthread_mutex_t lock[NBUCKET]; // declare a lock

static void put(int key, int value) {
    int i = key % NBUCKET;

    // is the key already present?
    struct entry *e = 0;
    for (e = table[i]; e != 0; e = e->next) {
        if (e->key == key)
            break;
    }
    if (e) {
        // update the existing key.
        e->value = value;
    } else {
        // the new is new.
        pthread_mutex_lock(&lock[i]);
        insert(key, value, &table[i], table[i]);
        pthread_mutex_lock(&lock[i]);
    }
}

int main(int argc, char *argv[]) {
    pthread_mutex_init(&lock[NBUCKET], NULL);
    ......
}
```
```bash
$./ph 10
100000 puts, 0.684 seconds, 146180 puts/second
......
```

这样就足以跑过`ph_fast`的测试了（其实如果想偷鸡直接将NBUCKET改成一个大点的质数减少一下hash冲突就能用大锁跑过测试）

还有更细粒度的加锁方案，就是对链表（entry）的节点加锁，不过通常设计比较好的hashmap的拉链，链表不会很长。这里就不这么加了。

还有一种链表并发的方案就是rcu，不过这个是在内核态实现的，想要在userspace模仿一个有点难。

### Barrier

这个也是在Unix/linux真机下面，练习一下条件变量。这个barrier是用`pthread_cond`实现的，`a point in an application at which all participating threads must wait until all other participating threads reach that point too. `

让执行完的线程wait(睡眠)，然后面足条件的时候调用signal唤醒，这里全部唤醒用broadcast就好了。

注意使用条件变量的时候需要加锁，参考Three Easy Piecies第30章。某些情况下signal可以不加锁，但是wait必须要加锁。`hold the lock when calling signal`，不然会发生race，书里给了一个例子：

```c
void thr_exit() {
  done = 1;
  Pthread_cond_signal(&c);
}

void thr_join() {
  if (done == 0) {
    Pthread_cond_wait(&c);
  }
}
```

这个时候如果一个thread执行`thr_join`，满足done为0，开始wait睡眠，但是在判断完if到执行wait睡眠的期间，`thr_exit()`开始执行唤醒全部全部线程，那么这个thread执行wait就会一直sleep醒不过来了（书里面是用的parent和child的例子）

```c
static void barrier() {
    // Block until all threads have called barrier() and
    pthread_mutex_lock(&bstate.barrier_mutex);
    bstate.nthread++;
    if (bstate.nthread < nthread) {
        pthread_cond_wait(&bstate.barrier_cond, &bstate.barrier_mutex);
    } 
    pthread_mutex_unlock(&bstate.barrier_mutex);

    // 这个锁也要加上，不然唤醒的时候如果有线程睡眠会睡死
    pthread_mutex_lock(&bstate.barrier_mutex);
    if (bstate.nthread == nthread) {
        bstate.nthread = 0;
        bstate.round++;
        pthread_cond_broadcast(&bstate.barrier_cond);
        pthread_mutex_unlock(&bstate.barrier_mutex);
    }
}
```

这样`./barrier 100`就pass了。如果我们不加mutex锁，那么`./barrier 100`是会一直卡在那里结束不了的。如果我们执行strace看一下`strace ./barrier 100`，会发现卡在这个地方

```c
futex(0x7f0a8b7a1910, FUTEX_WAIT_BITSET|FUTEX_CLOCK_REALTIME, 219673, NULL, FUTEX_BITSET_MATCH_ANY) = ? ERESTARTSYS (To be restarted if SA_RESTART is set)
--- SIGWINCH {si_signo=SIGWINCH, si_code=SI_KERNEL} ---
futex(0x7f0a8b7a1910, FUTEX_WAIT_BITSET|FUTEX_CLOCK_REALTIME, 219673, NULL, FUTEX_BITSET_MATCH_ANY
```

用`valgrind --tool=helgrind ./barrier 2`检查了一下，应该这样也是可以的，都加到一把锁里面

```c
static void barrier() {
    // Block until all threads have called barrier() and
    pthread_mutex_lock(&bstate.barrier_mutex);
    bstate.nthread++;
    if (bstate.nthread < nthread) {
        pthread_cond_wait(&bstate.barrier_cond, &bstate.barrier_mutex);
    } 
    if (bstate.nthread == nthread) {
        bstate.nthread = 0;
        bstate.round++;
        pthread_cond_broadcast(&bstate.barrier_cond);
        pthread_mutex_unlock(&bstate.barrier_mutex);
    }
}
```

## 小结

这次看题目本来以为很简单的，用户态thread就照着内核线程抄一个呗，剩下两个就是用一下pthread和锁呗。然后写起来debug就花了很久的时间，对寄存器不太熟，频繁翻车。然后折腾了两天才整出来

```bash
$ make grade
uthread: OK (2.5s) 
== Test answers-thread.txt == answers-thread.txt: OK 
== Test ph_safe == make[1]: Entering directory '/home/zhixi/codes/xv6-labs-2021'
gcc -o ph -g -O2 -DSOL_THREAD -DLAB_THREAD notxv6/ph.c -pthread
make[1]: Leaving directory '/home/zhixi/codes/xv6-labs-2021'
ph_safe: OK (8.0s) 
== Test ph_fast == make[1]: Entering directory '/home/zhixi/codes/xv6-labs-2021'
make[1]: 'ph' is up to date.
make[1]: Leaving directory '/home/zhixi/codes/xv6-labs-2021'
ph_fast: OK (19.7s) 
== Test barrier == make[1]: Entering directory '/home/zhixi/codes/xv6-labs-2021'
gcc -o barrier -g -O2 -DSOL_THREAD -DLAB_THREAD notxv6/barrier.c -pthread
make[1]: Leaving directory '/home/zhixi/codes/xv6-labs-2021'
barrier: OK (2.7s) 
== Test time == 
time: OK 
Score: 60/60
```

## 链接

<https://pdos.csail.mit.edu/6.S081/2020/labs/thread.html>   

[How do you pass a function as a parameter in C?](https://stackoverflow.com/questions/9410/how-do-you-pass-a-function-as-a-parameter-in-c)  

[Go Map in Actions](https://go.dev/blog/maps)  
<https://github.com/golang/go/blob/master/src/sync/map.go>  
<https://pkg.go.dev/sync?utm_source=godoc#Map>   
[Go 1.9 sync.Map揭秘](https://colobu.com/2017/07/11/dive-into-sync-Map/) 鸟窝大大写的sync.Mutex介绍  

[Operating Systems Three Easy Pices](https://pages.cs.wisc.edu/~remzi/OSTEP/)，中文就是那个蓝皮的操作系统导论  
第29章，讲并发链表、并发队列和并发散列表  
第20章，讲条件变量  
---
title: Linux内存管理之OOM(Out of Memory)
date: 2021-09-14 21:30:11
abbrlink: 'linux-out-of-memory'
tags:
- Linux
- OOM
---
之前在Earlyoom和oomd那一篇开始写OOM的介绍写得太长了，拿出来单独讲一下Overcommit和OOM killer
<!-- more -->

## OOM killer

如果你想关闭OOM killer

具体实现见：[oom_kill.c](https://github.com/torvalds/linux/blob/master/mm/oom_kill.c)。实现比较复杂，这里挑几个重要的功能简单说一下：

`oom_kill_allocating_task`这个函数是决定OOM时kill哪个进程

三个重要参数：`oom_adj`， `oom_score`， `oom_score_adj`位于`/proc/PID`

`oom_adj`是进程被kill的几率

### 需要注意的几个问题

不仅是在系统整体的内存耗尽才会触发OOM-killer工作，比如使用了cgroup

Linux是允许申请比可用内存更多的内存的，这被称为OverCommit
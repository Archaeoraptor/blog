---
title: Linux添加系统调用：在Archlinux下编译内核
date: 2022-03-22 19:59:43
tags:
- linux
- syscall
- archlinux
abbrlink: 'add-a-syscall-to-linux-kernel'
categories:
- Linux&Unix
---
今天手痒玩一下Archlinux的编译内核，顺便观摩一下Arch内核打包是什么样的（不知道用什么举例子就添加个syscall算了）
<!-- more -->

体验还不错，Archlinux编译内核啥的还都挺简单的（相比于LFS），而且打包也方便。以后不开qemu了，直接裸机编译内核玩喽。  

## 下载和准备编译环境

Archlinux编译内核有两种方式，一种是传统的编译方式（玩过LFS或者gentoo的应该很熟悉），另一种是ABS（Arch Build System，可以打一个AUR包供他人使用）。其他发行版的读者请跳过ABS，只看手动编译即可。

第一种和通常的手工编译内核没什么区别，这里先演示添加系统调用并手动打包。（如果不是为了打包其实直接手动编译更方便点）下载内核源码并解压，内核版本和解压路径换成你自己想要的（如果不想打包生成patch文件，不需要`git init`）

```bash
wget https://mirrors.tuna.tsinghua.edu.cn/kernel/v5.x/linux-5.16.15.tar.xz
mkdir ./linux-zhixi
tar xf linux-5.16.15.tar.xz --directory=./linux-zhixi --strip-components 1
cd linux-zhixi
# if you want to create a patch for packaging
# git init
```

## 添加系统调用

接下来我们添加一个系统调用，统计开机到现在用了多少秒。
修改`arch/x86/entry/syscalls/syscall_64.tbl`系统调用表，仿照上面的格式加上一行，

```tbl
548 common  uptime          sys_getuptime
```

然后增加系统调用，随便找个位置，比如`kernel/time/getuptime.c`， 但是为了不把kernel目录搞乱，我自己建一个`zhixi`的目录放`getuptime.c`

```c
/*
 * sys_getuptime - get how many seconds have elapsed since boot
 * @secs: return the value of secs corresponding to jiffies
 */
#include <linux/jiffies.h>
SYSCALL_DEFINE1(getuptime, unsigned int __user *, secs)
{
        unsigned long kjiffies = (get_jiffies_64() - INITIAL_JIFFIES) / 1000;
        unsigned int ksecs = jiffies_to_secs(kjiffies);
        copy_to_user(secs, &ksecs, sizeof(ksecs));
        return 0;
}
```

为了在编译内核的时候编译这个文件，在`zhixi`目录下增加`Makefile`

```makefile
obj-y := getuptime.o
```

然后给源码根目录的Makefile加上`zhixi/`

```makefile
core-y += kernel/ mm/ fs/ ipc/ security/ crypto/ block/ zhixi/
```

然后编辑`include/linux/syscalls.h`头文件加上自己的系统调用：

```c
/* zhixi/getuptime.c */
asmlinkage long sys_getms_sinceboot(unsigned int __user *secs);
```

一个简单的系统调用就加好了。

## 手动编译内核

编译内核的时候可以直接

```bash
make menuconfig
```

## 试试看



## 如果想打包

下面是将你自己改过的内核打包，如果你不用Archlinux

```bash
pacman -S base-devel
```

### 给kernel打patch

在PKGBUILD中打patch请参考：[Patching_packages_(简体中文)#应用补丁](https://wiki.archlinux.org/title/Patching_packages_(简体中文)#应用补丁)  

### 编译和打包



## 链接

[ArchWiki Patching pacakages](https://wiki.archlinux.org/title/Patching_packages)   
[Kernel/Traditional compilation](https://wiki.archlinux.org/title/Kernel/Traditional_compilation)   
[Arch Build System (简体中文)](ttps://wiki.archlinux.org/title/Arch_Build_System_(简体中文))   
[X86_64 架构增加一个系统调用](https://biscuitos.github.io/blog/SYSCALL_ADD_NEW_X86_64/#header)  
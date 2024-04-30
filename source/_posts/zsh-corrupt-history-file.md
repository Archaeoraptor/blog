---
title: 记一次ext4文件系统损坏，zsh corrupt history file
date: 2024-04-26 13:54:10
tags:
- zsh
abbrlink: 'zsh-corrupt-history-file'
---
zsh: corrupt history file /home/zhixi/.histfile
<!-- more -->
Vmware虚拟机里面的那台arch因为用的ext4，然后经常随手强行关机就坏了。

已经挂载上了fsck不太好修。

```bash
fsck from util-linux 2.40
e2fsck 1.47.0 (5-Feb-2023)
/dev/sda2 is mounted.
e2fsck: Cannot continue, aborting.
```

要用fsck需要，改设置CD-ROM启动，然后chroot挂载，然后fsck

但是zsh history文件比较简单，直接修复一下就可以了。网上有很多教程，比如


```bash
cd ~
mv .zsh_history .zsh_history_bad
strings .zsh_history_bad > .zsh_history
fc -R .zsh_history
rm ~/.zsh_history_bad
```

我把zsh的历史命令设置存到`~/histfile`了，所以应该是

```bash
mv ~/.histfile ~/.histfile_bad
# 用strings命令提取
strings ~/.histfile > ~/.histfile_bad
fc -R ~/.histfile
rm ~/.histfile_bad
```

但是这样会有一个问题，中文乱码。即使使用`strings -eS .zsh_history`，能读取到中文命令，中文也会乱码。像下面这样

```bash
git log
code 练�僰��总僲��-⃥�.go
clear
```

## 链接

[How to fix a corrupt zsh history file](https://shapeshed.com/zsh-corrupt-history-file/)  

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
strings ~/.histfile_bad > ~/.histfile
fc -R ~/.histfile
rm ~/.histfile_bad
```

但是这样会有一个问题，中文乱码。即使使用`strings -eS .zsh_history`，能读取到中文命令，中文也会乱码。像下面这样

```bash
code 练�僰��总僲��-⃥�.go
```

试过几种编码都不行，查了一下发现zsh有自己的编码格式。可以这样解码：

```c
#define Meta ((char) 0x83)

#define _GNU_SOURCE
#include <stdio.h>
#include <stdlib.h>

/* from zsh utils.c */
char *unmetafy(char *s, int *len)
{
  char *p, *t;

  for (p = s; *p && *p != Meta; p++);
  for (t = p; (*t = *p++);)
    if (*t++ == Meta)
      t[-1] = *p++ ^ 32;
  if (len)
    *len = t - s;
  return s;
}

int main(int argc, char *argv[]) {
  char *line = NULL;
  size_t size;

  while (getline(&line, &size, stdin) != -1) {
    unmetafy(line, NULL);
    printf("%s", line);
  }

  if (line) free(line);
  return EXIT_SUCCESS;
}
```

又查了一下在终端中可以直接用`fc -l`查看，`fc 1 1000`查看1到1000条的记录。那就好办了。`$HISTFILE`变量为历史存储的位置，`SAVEHIST`变量为保存历史的条数。

所以可以写成这样：

```zsh
cp `$HISTFILE` ~/.histfile_bad
# 用strings命令提取会乱码
# strings `$HISTFILE` > ~/.histfile
# 用zsh内置fc命令自带的解码可以读取
fc -l 1 $SAVEHIST | awk '{print $2}' > ~/.histfile_tmp
cp ~/.histfile_tmp $HISTFILE
fc -R $HISTFILE
rm ~/.histfile_bad ~/.histfile_tmp
# 运行zsh没有报错就可以了
zsh
```

## 链接

[How to fix a corrupt zsh history file](https://shapeshed.com/zsh-corrupt-history-file/)  
[Re: Fw: ZSH history file VS. UTF-8 data](https://www.zsh.org/mla/users/2011/msg00154.html)  

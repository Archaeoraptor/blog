---
title: Git存储结构与实现
date: 2023-11-13 16:00:07
tags:
- git
categories:
- 笔记
abbrlink:
- 'git-structure'
---
突然发现git还挺有意思的，实现很像kv存储文件系统。
<!-- more -->

有一天博客的repo在vscode侧栏source control显示不出更改文件和详情了，只有一个几个更改的文件个数，但是其他的repo又是好的。用sublime merge和`git diff`,`git status`等其他工具和命令也都是好的。（别问我为什么不用sublime merge，我现在被迫）于是研究了半天`.git`文件夹和git原理试图搞清哪出问题了。（最后发现是submodule的问题）之前在hackergame2023游玩的时候有道题是从 git repo 里找flag，然后又去拆`.git`文件夹。看着看着发现git有点意思。

先来看一下git的目录结构，`.git`目录下面文件大概长这样：

```bash
>cd .git
>tree -L 1
.
├── branches
├── COMMIT_EDITMSG
├── config
├── description
├── FETCH_HEAD
├── HEAD
├── hooks
├── index
├── info
├── logs
├── objects
├── ORIG_HEAD
├── packed-refs
└── refs
```

用`ncdu`看一下文件的大小，发现Objects是很大的

`objects`下面有info和pack，以及存放的所有对象

```bash
>cd objects && ls
03  15  27  35  49  5e  64  6c  7b  84  99  ac  bf  c6  d3  e0  ee  fd
0b  17  2a  37  4e  60  65  6e  7c  85  9c  ba  c0  c7  d4  e4  f0  fe
0d  20  2d  40  53  62  69  72  7f  87  a4  bc  c1  c9  dd  ea  f5  info
0f  21  34  43  55  63  6b  74  81  92  a9  bd  c2  d2  df  ed  f6  pack
```

git用sha1值存储，其他的两个数字或字母开头的文件名是sha1前两位，目录中文件名是sha1值，文件采用zlib压缩格式

```bash
~/codes/blog/.g/objects master ?1 ❯ tree                               7s
.
├── 03
│   └── 4c85b33ffa84adfb9f44d3c38fe13915e18a7c
├── 0b
│   └── 54054172c2050751407e0e9ae9a6c67436822c
├── 0d
│   ├── 0bc9b8cf7b882a3e5dfcd0790f09bc3482e8fe
│   ├── 48c09ea03af3c3f6667be46ed57514e257acfc
│   └── a1ba91c7ead0c0bb4e020a89b8a81ccaec137d
```

今年的hackergame的[git题目](https://github.com/USTC-Hackergame/hackergame2023-writeups/blob/master/official/Git%20Git!/README.md)就是从这里面找flag，撤销commit后在objects下面还有残留数据，于是从objects下的zlib文件后解压后查找flag即可。（文件名的hash可以从git reflog中获取）

解压可以用gzip或者openssl

```bash
zlibd() (printf "\x1f\x8b\x08\x00\x00\x00\x00\x00" | cat - "$@" | gzip -dc)
zlibd 97126e45aebd4c938edc8e11c28cef347cd317 | hexdump -C

# 或者用pigz
unpigz < 97126e45aebd4c938edc8e11c28cef347cd317 | hexdump -C
```

或者蟒蛇

```python
import zlib
s = '...'
z = zlib.uncompress(s)
with open('yourfile', 'w') as f:
    f.write(z)
```

refs目录比较简单，是对obejects的reference，其中origin的HEAD为远程分支。packed-refs文件和refs作用一样。

```bash
~/codes/blog/.git master ?1 ❯ tree refs        
refs
├── heads
│   └── master
├── remotes
│   └── origin
│       ├── HEAD
│       └── master
└── tags
```

config文件为git的配置，index是索引, logs是提交日志，hooks下面是一堆bash脚本

## 原理

git的对象包括blob，tree，commit，tag，blob是文件，tree是目录，在上面的目录结构中都已经看过了。还有一些二进制文件，推荐使用`git-lfs`

git的存储方式是通过类似inode的指针，一般是一个树状，但是commit和branch会变成一个比较复杂的图。

![pointer](git-structure/image.png)  

![git](git-structure/image-1.png)  

- add操作

`git add`会把文件添加到缓冲区（staged）

- commit操作

`git commit`会添加缓冲区的版本  
如果`git commit -a`会直接跳过缓冲区（跳过git add），直接添加untracked的版本

- rm操作

rm操作把文件从tracked file list中移除（但是Object中还有历史版本，可以找到）。还没有staged的文件可以直接`git rm`，但是如果已经staged或者已经commit添加到snapshot中，就需要`git rm -f`。

如果直接rm，会把文件也删除，大多数情况下是不小心把一堆编译产物或者二进制文件加到缓冲区去了，这时候要用

```bash
git rm --cached yourfile
```

- mv操作

git中没有类似文件系统的重命名，mv操作相当于删除后再次添加

```bash
git mv oldname newname
# 相当于
mv oldname newname
git rm oldname
git add newname
```

### git是增量存储吗

中文网上大多数资料（比如不靠谱的某乎某SDN某思否）想当然的说git是增量存储的。但是，Pro Git和 Github Blog上说了，**Commits are snapshots, not diffs**。当然是选择相信Github Blog啦，如果不信自己建一个repo commit一下试试看看上一节的`.git`目录里的Object有什么变化就知道了。上一接我们已经看过Objects下面的blob了，并不是增量的。



git采用的保存方式是快照（snapshot），比较像

git的增量，是当每次更改的时候


### 远程

- push, pull, clone


## 实现

初版git的实现比较简单，代码量也比较少，可以观摩一下。

```bash
git clone https://github.com/git/git.git
git checkout e83c5163316f89bfbde7d9ab23ca2e25604af290
```

初版代码量很少，没有今天常用的`git add`和`git commit`等功能。直接`make`会报错，需要修改

## link

[git](https://github.com/git/git)  
[Pro Git](https://git-scm.com/book)  
[Commits are snapshots, not diffs](https://github.blog/2020-12-17-commits-are-snapshots-not-diffs/)  
[Git’s database internals I: packed object store](https://github.blog/2022-08-29-gits-database-internals-i-packed-object-store/)  
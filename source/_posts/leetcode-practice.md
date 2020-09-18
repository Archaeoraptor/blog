---
title: Leetcode刷题
tags:
  - leetcode
categories:
  - 杂七杂八
abbrlink: 6aa3
date: 2019-10-13 19:40:29
---
我就每天当瓜子嗑了，不刷题练练手真的要成一个废人了
<!--more-->

生活所迫，开始刷题练手

装上了VSCode，开始在教研室摸鱼式刷题，再次感受到了VSCode本质上是一个浏览器。

现在还有点纠结拿什么语言刷，哎，不行就先把那本算法竞赛入门经典刷一遍吧。大一学的数据结构都快忘光了。(⊙﹏⊙)

## VSCode RemoteDevelopment插件连接wsl

教研室分给我的坑位上那台即将报废回收破主机是个14年的i5，并没有独显，虚拟机和双系统是跑不太动了，而且我想用Docker，打开hyper-v之后VMware也不兼容，就弃坑VM了。
开始想在win10下装clang和gcc，后来发现用VSCode直接上wsl更省事，安装Remote Development插件就行了
在wsl新建目录输入`code-insiders .`或者`code .`
或者在VSCode里按F1，选择`Remote-WSL: New Window`，再打开文件夹就行了，看到左下角有个绿色的wsl标志就可以了
git还会有点小问题，要禁用换行符自动转换，不然win下面和linux下面不一样每次git的repo明明没改动，却显示新的变化，问题看[这里](https://code.visualstudio.com/docs/remote/wsl)

```git
git config --global core.autocrlf false
```

wsl的Debian应该是buster版本的，镜像源选buster（Debian10）

wsl里cd进`\mnt`目录就能和win10交换文件，win10下的wsl文件目录在`C:\Users\Username\AppData\Local\Packages\CanonicalGroupLimited.UbuntuonWindows_79rhkp1fndgsc\LocalState\rootfs`

wsl里的shell换成oh my zsh，哎，舒服
配置问题请参考[官方文档](https://code.visualstudio.com/docs/cpp/config-wsl)，可以装code runner插件像IDE一样直接运行
[看这里](https://printempw.github.io/vscode-c-cpp-configuration-for-acm-oj/)
更新，由于巨硬的VSCode频繁更新，json配置格式乱改，折腾来折腾去非常麻烦，请直接使用别人打包好的便携版或者直接转投Clion（有教育邮箱的我就转投Clion了，JB全家桶真香）
据说wsl2已经在20H1预览版上了，但是win10这个样子我又不敢乱更新，等等看吧，观望一下再决定黑巨硬还是微软真香。

## leetcode

看了一下leetcode对语言的支持
C++：g++ 8.2，C++14 标准，C：gcc 8.2

[69] sqrt(x)

二分法或者牛顿法

[231] Power of Two

不断除以二就行了，或者按位与

```python
num & (num-1)
```

[193] Valid Phone Numbers
美国地区电话号码，直接用正则就好了`^(\d{3}-|\(\d{3}\) )\d{3}-\d{4}$`

## 其他

努力不写不求甚解的谭++
>有一些很简单的鉴别方法 比如创建obj用的Type()而不是Type{}, 到处是类型声明, 各种子类继承叠罗汉 都是谭++的标志

### alloca和malloc函数

C/C++里面的alloca函数动态分配内存, 从栈申请内存，自动释放。alloca是最快的动态内存分配，对应的汇编只有一条指令 sub rsp, size。
C还支持可变长数组（variable-length array， VLA）（C99以后）
malloc从堆中申请内存，需要手动释放（free）

### C++和STL

等待填坑

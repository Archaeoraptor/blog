---
title: Linux的启动过程（顺便优化一下开机速度）
date: 2021-09-14 19:26:18
abbrlink: 'speedup-booting'
tags:
- boot
- Linux
---
您的开机已打败8%的用户，可是，为什么要在乎启动速度呢？不关机不就行了。
<!-- more -->

## 加快开机速度


### 懒得折腾

不关机就行了  
grub换成systemd-boot  
关掉开机自检（有风险，不推荐）  
机械换成SD卡或固态   

### 折腾一下

裁剪内核，只保留

#### 直接跑在内存上

[把操作系统linux运行在内存上](https://www.bo-song.com/把操作系统linux运行在内存上/)  
https://en.wikipedia.org/wiki/List_of_Linux_distributions_that_run_from_RAM  

Archlinux可以直接用ramroot。

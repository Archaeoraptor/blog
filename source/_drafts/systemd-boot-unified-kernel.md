---
title: 用systemd-boot做bootloader
date: 2022-04-10 13:41:03
tags:
- systemd-boot
- linux
- bootloader
abbrlink: 'systemd-boot-unified-kernel'
categories:
- Linux&Unix
---
之前
底裤又没了一个耶（但是它启动快啊，grub好臃肿
<!-- more -->

## 从Grub迁移到systemd-boot

grub有好看的主题，也能方便的在开机的时候选内核版本、进btrfs的sanpshot回滚

```bahs
$sudo bootctl --path=/boot install
File system "/dev/block/259:1" has wrong type for an EFI System Partition (ESP).
```

## 浅谈启动流程

先来看看最简单的单片机启动，没有内核也没有BIOS和UEFI，非常的简单。

## 那安全启动呢

我没加密boot分区，也没配置安全启动。



## 链接

[[原创]浅析安全启动（Secure Boot） 
](https://bbs.pediy.com/thread-260399.htm)  

[Writing an x86 "Hello world" bootloader with assembly](http://50linesofco.de/post/2018-02-28-writing-an-x86-hello-world-bootloader-with-assembly)

### systemd-boot参考

https://wiki.archlinux.org/title/Arch_boot_process 
https://wiki.archlinux.org/title/Systemd-boot  
https://wiki.archlinux.org/title/EFI_system_partition  
https://wiki.archlinux.org/title/Unified_kernel_image  
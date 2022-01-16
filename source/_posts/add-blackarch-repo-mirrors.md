---
layout: posts
title: 给Archlinux添加blackarch仓库镜像源
date: 2020-11-12 22:08:30
tags:
  - blackarch
abbrlink: 'add-blackarch-repo-mirrors'
categories:
- 不务正业系列
---

更新：BlackArch镜像源的很多包质量很差，请谨慎添加，必要时请修改PKGBUILD或自行打包。
<!-- more -->

## 更新和劝退（只是想添加blackarch源的请直接看第二节）

虽然BlackArch的包很多，但是.......

BlackArch的打包质量很差（甚至有些包可能比AUR更差），不少包都是用工具直接转的，而且更要命的是很多包缺乏维护（这对于Arch系这些更新频繁的包可不太妙）

会发生一些离谱的事情，比如这种：https://freenode.logbot.info/archlinux-cn/20210518#c8047029

一些活跃的第三方源，比如archlinuxcn，arch4edu，处理问题比较及时，质量也比AUR的包更放心。一般的AUR helper，比如yay, pikaur都喜欢把添加的源里的包优先级放到AUR前面。`sudo pacman -Syyu`也会把本地新的包降级。BlackArch里面很多万年不更新也没人维护的老包，会很麻烦。

比如neo4j-community，AUR比blackarch源里的新，直接用pacman或者yay等AUR Helper安装就会装一个远古的老版本     
（不是3.X版本稳定的原因，单纯就是没人维护，[官网的版本](https://neo4j.com/download-center/?ref=subscription#community)现在3.5.28是20 April 2021发布的，BlackArch的官方源更新是在一[2020年3月](https://github.com/BlackArch/blackarch/commit/ab4980a36cd471ca4f4f4ca1c17d6b7e5e6daae9)）

```log
aur/neo4j-community 4.2.2-1 (+63 0.40) (Out-of-date: 2021-06-06) (Installed)
    A fully transactional graph database implemented in Java
blackarch/neo4j-community 3.5.14-1 (144.4 MiB 165.9 MiB) (Installed: 4.2.2-1)
    A fully transactional graph database implemented in Java
```

主动安装AUR里的可以这样：`yay neo4j-community`然后`Packages to install`选项手动选AUR里的。或者你手动下载AUR里的PKGBUILD然后`makepkg`自己build。**再或者直接不要blackarch这个第三方源了（推荐）**，`/etc/pacman.d`里注释掉。

不然每次更新还要忽略一堆`warning: neo4j-community: local (4.2.2-1) is newer than blackarch (3.5.14-1)`。

neo4j这种单个没啥影响的包更新迟缓倒也没啥大问题，问题是BlackArch的包经常会影响community/extra的基础包，比如plasma这个包（某一天某个群友亲历，我由于装的是plasma-meta逃过一劫）

KDE用户注意：如果你添加了BlackArch源，按照[ArchWiki上的KDE页面](https://wiki.archlinux.org/title/KDE#Plasma)添加plasma元组

```bash
sudo pacman -Syu plasma
```

不仔细看plasma是安装成功了，也没报错。然后就掉到Blackarch的plasma这个包的坑里的,[plamsa](https://github.com/BlackArch/blackarch/blob/master/packages/plasma/PKGBUILD)是BlackArch里面一个python写的一个工具的包，跟KDE Plasma一点关系都没有。

没想到吧，我也没想到。因为**按照ArchLinux的VSC打包规范**，[plasma](https://github.com/plasma-disassembler/plasma)这个东西的包名**应该叫做 plasma-git**

你得这样安装：

```bash
yay -Syu extra/plasma.
```

BlackArch仓库人手不够打包质量差、更新不及时我都能理解，能跟extra仓库冲突了就离谱。

下面传肥猫领袖重要讲话：

```
Felix Yan, [05.02.21 01:08]
想起来我当年研究过 blackarch 的仓库

Felix Yan, [05.02.21 01:09]
想把里面的工具往主仓库搬

Felix Yan, [05.02.21 01:09]
结果真找不到几个能搬的

Felix Yan, [05.02.21 01:09]
大部分都是坑工具……

（里面有很多质量堪忧或者版权不明白的工具，不适合放到一个正常发行版

质量好的工具我当年整过一波，该打的大部分都打了
```

或许单独装个kali是个不错的选择。平时日用好像Parrot os也不错。我反正Arch够用了（我又不干坏事）。

以下为原内容，写于 2020.11 目测有效，不保证该方法以后可能过时，建议自行翻阅官方文档。

## 导入仓库和设置国内镜像

### 导入仓库

按照官方文档来：

下载官网的脚本

```bash
curl -O https://blackarch.org/strap.sh
```

修改权限：

```bash
chmod +x strap.sh
```

运行

```bash
./strap.sh
```

然后应该就好了

看看是不是装好了：

```bash
pacman -Sg | grep blackarch
```

然后更新一下

```bash
sudo pacman -Syyu
```

这时候应该会给你安装`blackarch-keying`这个包，然后就可以用了

### 镜像源

修改`/etc/pacman.conf`，在最后加上

```conf
[blackarch]
Server = http://mirrors.uestc.cn/blackarch/$repo/os/$arch
```

Server请改成你在那里提供服务而且网速比较快的，比如tuna的

```conf
[blackarch]
Server = https://mirrors.tuna.tsinghua.edu.cn/blackarch/$repo/os/$arch
```

## 为何要写这么水的一篇博客

我之前谷歌随手搜出了ustc的镜像使用帮助，然后就开始了飘红报错，试了一圈中文的解决方案，并没有用。
（当然中文博客基本都过时了，还是有很多好好写的，不止CSDN,比如这个：[用 Arch Linux 打造完美渗透环境](https://weepingdogel.github.io/posts/用arch_linux打造完美渗透环境/)）

报错：

```bash
error: blackarch: key "F9A6E68A711354D84A9B91637533BAFE69A25079" is unknown
:: Import PGP key F9A6E68A711354D84A9B91637533BAFE69A25079? [Y/n] Y
Y
error: key "F9A6E68A711354D84A9B91637533BAFE69A25079" could not be looked up remotely
:: Synchronizing package databases...


error: blackarch: key "F9A6E68A711354D84A9B91637533BAFE69A25079" is unknown
:: Import PGP key F9A6E68A711354D84A9B91637533BAFE69A25079? [Y/n] 
error: key "F9A6E68A711354D84A9B91637533BAFE69A25079" could not be looked up remotely
error: failed to update blackarch (invalid or corrupted database (PGP signature))
error: failed to synchronize all databases

```

发现中文的几个解决方案全都过时了。安装`blackarch-keying`导入key会产生先有鸡还是先有蛋的问题。

```bash
$ sudo pacman -S blackarch-keyring
error: blackarch: key "F9A6E68A711354D84A9B91637533BAFE69A25079" is unknown
:: Import PGP key F9A6E68A711354D84A9B91637533BAFE69A25079? [Y/n] Y
error: key "F9A6E68A711354D84A9B91637533BAFE69A25079" could not be looked up remotely
error: database 'blackarch' is not valid (invalid or corrupted database (PGP signature))
```

转了一圈还是翻了官方文档，世界瞬间清净了。

```bash
[+] installing blackarch keyring...

loading packages...
resolving dependencies...
looking for conflicting packages...

Packages (1) blackarch-keyring-20140118-3

Total Installed Size:  0.04 MiB

:: Proceed with installation? [Y/n] 
(1/1) checking keys in keyring                                         [#######################################] 100%
(1/1) checking package integrity                                       [#######################################] 100%
(1/1) loading package files                                            [#######################################] 100%
(1/1) checking for file conflicts                                      [#######################################] 100%
:: Processing package changes...
(1/1) installing blackarch-keyring                                     [#######################################] 100%
==> Appending keys from blackarch.gpg...
gpg: public key DB323392796CA067 is 3037 days newer than the signature
gpg: public key CF66D153D884358F is 16 seconds newer than the signature
==> Locally signing trusted keys in keyring...
  -> Locally signing key A0917C4147A37007CB54C1CFD295AA940EFDDF62...
  -> Locally signing key 4345771566D76038C7FEB43863EC0ADBEA87E4E3...
  -> Locally signing key CBA3C7D4798912702DCF568E67D8BDF42AD93F4E...
  -> Locally signing key 8F9A9793CB8591147C2EC70566E0CDBD1E01F333...
==> Importing owner trust values...
gpg: inserting ownertrust of 4
gpg: setting ownertrust to 4
gpg: setting ownertrust to 4
gpg: setting ownertrust to 4
==> Updating trust database...
gpg: key 1EB2638FF56C0C53: no user ID for key signature packet of class 10
gpg: key 1EB2638FF56C0C53: no user ID for key signature packet of class 10

gpg: public key DB323392796CA067 is 3037 days newer than the signature
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: public key CF66D153D884358F is 16 seconds newer than the signature
gpg: depth: 0  valid:   1  signed:  56  trust: 0-, 0q, 0n, 0m, 0f, 1u
gpg: depth: 1  valid:  56  signed:  81  trust: 1-, 0q, 0n, 55m, 0f, 0u
gpg: depth: 2  valid:  77  signed:  26  trust: 77-, 0q, 0n, 0m, 0f, 0u
gpg: next trustdb check due at 2020-12-03
:: Running post-transaction hooks...
(1/1) Arming ConditionNeedsUpdate...
==> Appending keys from archlinuxcn.gpg...
==> Appending keys from archlinux.gpg...
==> Appending keys from blackarch.gpg...
==> Appending keys from endeavouros.gpg...
==> Locally signing trusted keys in keyring...
  -> Locally signing key 57957BAD5D038B07C28EF49A15B26377262268C1...
==> Importing owner trust values...
==> Disabling revoked keys in keyring...
  -> Disabling key 8F76BEEA0289F9E1D3E229C05F946DED983D4366...
==> Updating trust database...
gpg: next trustdb check due at 2020-12-03

[+] keyring installed successfully
[+] updating package databases
:: Synchronizing package databases...
 core                                      129.9 KiB  6.34 MiB/s 00:00 [---------------------------------------] 100%
 extra                                    1634.4 KiB  4.09 MiB/s 00:00 [---------------------------------------] 100%
 community                                   5.2 MiB  11.1 MiB/s 00:00 [---------------------------------------] 100%
 multilib                                  154.2 KiB  7.53 MiB/s 00:00 [---------------------------------------] 100%
 endeavouros                                14.2 KiB  0.00   B/s 00:00 [---------------------------------------] 100%
 archlinuxcn                              1354.5 KiB  11.0 MiB/s 00:00 [---------------------------------------] 100%
 blackarch                                   3.3 MiB   500 KiB/s 00:07 [---------------------------------------] 100%
 blackarch.sig                             566.0   B  0.00   B/s 00:00 [---------------------------------------] 100%
```

```bash
Packages (2) blackarch-keyring-20180925-2  perl-expect-1.35-6

Total Download Size:   0.07 MiB
Total Installed Size:  0.15 MiB
Net Upgrade Size:      0.00 MiB

:: Proceed with installation? [Y/n] Y
:: Retrieving packages...
 blackarch-keyring-20180925-2-any                                                                   18.1 KiB  0.00   B/s 00:00 [-----------------------------------------------------------------------------] 100%
 perl-expect-1.35-6-any                                                                             50.9 KiB  1698 KiB/s 00:00 [-----------------------------------------------------------------------------] 100%
(2/2) checking keys in keyring                                                                                                 [-----------------------------------------------------------------------------] 100%
(2/2) checking package integrity                                                                                               [-----------------------------------------------------------------------------] 100%
(2/2) loading package files                                                                                                    [-----------------------------------------------------------------------------] 100%
(2/2) checking for file conflicts                                                                                              [-----------------------------------------------------------------------------] 100%
:: Processing package changes...
(1/2) upgrading blackarch-keyring                                                                                              [-----------------------------------------------------------------------------] 100%
==> Appending keys from blackarch.gpg...
==> Locally signing trusted keys in keyring...
  -> Locally signing key A0917C4147A37007CB54C1CFD295AA940EFDDF62...
  -> Locally signing key 4345771566D76038C7FEB43863EC0ADBEA87E4E3...
  -> Locally signing key CBA3C7D4798912702DCF568E67D8BDF42AD93F4E...
  -> Locally signing key 8F9A9793CB8591147C2EC70566E0CDBD1E01F333...
==> Importing owner trust values...
==> Disabling revoked keys in keyring...
  -> Disabling key 5E210889BBB5C48500E0C4F9C75E985FF8B993B4...
==> Updating trust database...
gpg: key 1EB2638FF56C0C53: no user ID for key signature packet of class 10
gpg: key 1EB2638FF56C0C53: no user ID for key signature packet of class 10

gpg: public key DB323392796CA067 is 3037 days newer than the signature
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: public key CF66D153D884358F is 16 seconds newer than the signature
gpg: depth: 0  valid:   1  signed:  56  trust: 0-, 0q, 0n, 0m, 0f, 1u
gpg: depth: 1  valid:  56  signed:  81  trust: 1-, 0q, 0n, 55m, 0f, 0u
gpg: depth: 2  valid:  77  signed:  26  trust: 77-, 0q, 0n, 0m, 0f, 0u
gpg: next trustdb check due at 2020-12-03
(2/2) upgrading perl-expect                                                                                                    [-----------------------------------------------------------------------------] 100%
:: Running post-transaction hooks...
(1/2) Arming ConditionNeedsUpdate...
(2/2) Warn about old perl modules
:: Searching databases for updates...
:: Searching AUR for updates...
 -> python-pgspecial: local (1.11.10-2) is newer than blackarch (1.11.10-1)
 there is nothing to do
```

事实证明还是得看官方文档解决问题，以后直接看官方文档啊同志们！

一堆过时文档害人啊！

## 参考

1. <https://github.com/BlackArch/blackarch>  
2. <https://www.blackarch.org/downloads.html#install-repo>
3. [BlackArch Linux指南（中文翻译）](https://blackarch.org/blackarch-guide-zh.pdf)
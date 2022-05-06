---
title: 给老本子重装Arch：折腾一些灵车漂移的东西
date: 2022-05-04 21:08:14
tags:
- archlinux
abbrlink: 'reinstall-archlinux'
categories:
- Linux&Unix
---
换个文件系统不敢硬转，加上一堆想试试的新玩意，索性重装了
<!-- more -->

## 变动

### zram

一年多以前直接把内存从8G加到了32G，从此内存宽裕了。以前这个笔记本用的swap分区，然后默认开了zswap。后来偶尔要编译一些32G内存吃不消的东西，内存还是有点不够看，但我又不想把两根16G的条条卖了再买两根32G的
挺早就听说zram，然而只在内存吃紧的VPS和一个2G的树莓派上试过。这次索性给32G内存上了zram。休眠就不要了，自己买的SN550寿命要紧，减少点写入。

### 又多了一台Btrfs灵车

原来笔记本多次掉电强退，加上傻多戴ACPI的阴间问题，好几次以为关机成功，下次打开发现电都掉光了。两年前的时候刚装Arch

### systemd-boot

之前还留了一个windows双系统的，后来几乎完全生活在linux下，一年多都没打开过一次windows，然后就把windows扬了。既然不要windows那也不用上grub了。这次试试比臃肿的grub轻量的systemd-boot

### 试试sway

由于我的笔记本是1080p的，不用像折腾我那个台式的时候处理那些烦人的HiDPI的问题，用着wayland勉强能忍。用核显切到wayland之后确实丝滑流畅了，视频帧率都高了。不过Nvidia独显和sway八字不合，一些应用在wayland下也有各种小问题。还是装了一个KDE(x11)备用。

### 把tlp换成power-profiles-daemon

用了一天时间试了一下效果确实比原来的好。不过我的笔记本几乎全程插电用，外出的时候97w大电池+i3wm大概能顶六七个小时。重装完试了一天大概能撑9个小时左右。

### 上了pipewire

之前挺早就试用过pipewire，那个时候pipewire还有不少问题，现在基本没啥毛病了。

## 重装流程

之前装过四五次Arch，已经轻车熟路了。而且之前接受了Linux from Scratch的毒打，装起来倒是没啥困难，zram vs zswap， i3 vs sway 这样的选择困难症又犯了，纠结了很久。

```bash
iwctl
device list
station wlan0 scan
station wlan0 get-networks
station wlan0 connect Your-Wifi
exit

ping 1.1.1.1

timedatectl set-ntp true


mkfs.vfat -FAT32 -n BOOT /dev/nvme0n1p1
mkfs.btrfs /dev/nvme0n1p2
mount -t btrfs /mnt
btrfs sub create /mnt/@
btrfs sub create /mnt/@home
btrfs sub create /mnt/@snapshots
umount /mnt

# 这样就可以指定label了，不需要常常的UUID
btrfs filesystem label /dev/nvme0n1p2 ROOT

mount /dev/nvme0n1p2 /mnt -o subvol=@
mkdir /mnt/home
mkdir /mnt/var
mkdir /mnt/snapshots

mount /dev/nvme0n1p2 /mnt/home -o subvol=@home noatime
mount /dev/nvme0n1p2 /mnt/var -o subvol=@var noatime
mount /dev/nvme0n1p2 /mnt/snapshots -o subvol=@snapshots noatime

mkdir /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot

# 禁用COW
chattr +C /mnt/@var
chattr +C /mnt/@snapshots

## 然后可以安装包了
pacstrap /mnt base base-devel linux linux-headers linux-firmware intel-ucode
pacstrao /mnt sof-firmware  # 我这个笔记本要用的驱动，别的不用装
pacstrap /mnt btrfs-progs
pacstrap /mnt vim neovim iwd dhcpcd 

genfstab -U /mnt >>/mnt/etc/fstab
cat /mnt/etc/fstab

# 进入chroot
arch-chroot /mnt


echo 'LANG=en_US.UTF-8' >> /etc/locale.conf
echo 'zhixi' >> /etc/hostname
echo '127.0.0.1 localhost' >> /etc/hosts


passwd root

vim /etc/mkinitcpio.conf
# 在HOOKS=() 选项中添加btrfs
mkinitcpio -p linux

bootctl --path=/boot install

# 填好配置
vim /boot/loader/loader.conf
```

填写`arch.conf`, 我的大概长这样：

```conf
title          Arch Linux
linux          /vmlinuz-linux
initrd         /intel-ucode.img
initrd         /initramfs-linux.img
options        root=LABEL=ROOT   rw rootflags=subvol=@
```

一些小坑：

注意cfdisk分区的时候将512M的boot分区类型选为`EFI system`, 不然bootctl安装的时候会报`File system "/dev/black/259:1" has wrong type for an EFI System Partition (ESP).`

## 链接

https://askubuntu.com/questions/471912/zram-vs-zswap-vs-zcache-ultimate-guide-when-to-use-which-one/472227#472227

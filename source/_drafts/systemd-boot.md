---
title: 用Systemd-boot做Bootloader
date: 2022-04-10 13:41:03
tags:
- systemd-boot
- linux
- bootloader
abbrlink: 'systemd-boot'
categories:
- Linux&Unix
---
又研究了一下linux的启动流程，然后换成systemd-boot了。原先好看的grub主题就不要了，但是systemd-boot它启动快啊，grub好臃肿
<!-- more -->

## 从Grub迁移到systemd-boot

grub有好看的主题，也能方便的在开机的时候选内核版本、进btrfs的sanpshot回滚。但是我想尝试一下systemd-boot，grub看起来实在是太复杂了，我用不到那些功能。systemd-boot已经在systemd全家桶里面了，所以不需要额外安装。

systemd-boot一般要放在第一个EFI分区，而且直到今天systemd-boot和btrfs配合的还不是很好。我建了一个512M的EFI分区, 格式化为FAT32

注意需要EFI分区(EFI system partition, ef00)，不能是Linux filesystem(8300), 分区的Type如果是这样，grub是可以的，但是systemd-boot不认。用cgdisk重新分区的时候Type选ef00

```bash
$sudo bootctl --path=/boot install
File system "/dev/block/259:1" has wrong type for an EFI System Partition (ESP).
```

然后安装hook让每次更新内核的时候自动配置

```bash
yay -S systemd-boot-pacman-hook
```

systemd-boot是单文件启动的，配置文件放在`/boot/loader/entries`下面，可以根据不同的启动选项创建不同的配置文件，比如按照Arch Wiki的推荐创建一个`arch.conf`

```config
title Arch
linux /vmlinuz-linux
initrd /amd-ucode.img
initrd /initramfs-linux.img
options root=LABEL=ROOT rootflags=subvol=@ rw
```

title是名称，就是systemd-boot界面显示的那个。
linux选项是指定可执行的压缩内核，一般就`vmlinuz-linux`，如果是桌面用户装了其他内核应该能在`boot`目录下找到，比如`vmlinuz-linux-zen`

```bash
/boot❯ ls
amd-ucode.img                     initramfs-linux-lts.img              initramfs-linux-zen.img  vmlinuz-linux-lts
EFI                               initramfs-linux-xanmod-fallback.img  initramfs-linux.img      vmlinuz-linux-xanmod
initramfs-linux-fallback.img      initramfs-linux-xanmod.img           loader                   vmlinuz-linux-zen
initramfs-linux-lts-fallback.img  initramfs-linux-zen-fallback.img     vmlinuz-linux 
```

然后用initrd选项指定一下想要的initramfs镜像，装了amd或者intel的微码也要指定一下

最后options里面指定一下根分区和内核参数（如果用btrfs还有rootflags指定一下是哪个subvol）

ps: 在写rootflags的时候我们可以直接照抄grub生成的

```bash
sudo grub-mkconfig 2>/dev/null | grep vmlinuz
```

然后我们配置一下loader.conf, 指定默认启动使用`arch.conf`的配置

```config
default arch.conf
timeout 4
console-mode max
editor vim
```

现在boot目录的结构大概是这个样子：

```bash
❯ tree /boot
/boot
├── amd-ucode.img
├── EFI
│   ├── BOOT
│   │   └── BOOTX64.EFI
│   ├── Linux
│   └── systemd
│       └── systemd-bootx64.efi
├── initramfs-linux-fallback.img
├── initramfs-linux.img
├── loader
│   ├── entries
│   │   └── arch.conf
│   ├── loader.conf
│   └── random-seed
├── vmlinuz-linux
```

写好之后`bootctl status`检查一下，没有报错就可以重启了

```bash
❯ bootctl status
System:
     Firmware: UEFI 2.70 (American Megatrends 5.17)
  Secure Boot: disabled (setup)
 TPM2 Support: no
 Boot into FW: supported

Current Boot Loader:
      Product: systemd-boot 250.5-1-arch
     Features: ✓ Boot counting
               ✓ Menu timeout control
               ✓ One-shot menu timeout control
               ✓ Default entry control
               ✓ One-shot entry control
               ✓ Support for XBOOTLDR partition
               ✓ Support for passing random seed to OS
               ✓ Load drop-in drivers
               ✓ Boot loader sets ESP information
          ESP: /dev/disk/by-partuuid/c9d89129-dc40-0b4c-b9af-db4c0e9a47b8
         File: └─/EFI/SYSTEMD/SYSTEMD-BOOTX64.EFI
```

然后我们可以把grub卸了(留着吧，万一哪天生成rootflags参数或者又想用grub的好看主题了呢)

### 使用Unified Kernel image

就是把vmlinuz、ucode、initramfs的img都揉到一起做了一的大的可执行的内核镜像。当然这个可以手动制作，不过很多发行版提供了工具来生成它，比如Arch的mkinitcpio， 比如gentoo的genkernel, 再比如dracut

手动工作大概是这样的， 见： [sd-boot: add EFI boot manager and stub loader](https://github.com/systemd/systemd/commit/0fa2cac4f0cdefaf1addd7f1fe0fd8113db9360b#commitcomment-55422970), 就是用objcopy把那几个img文件拼起来  

```bash
$ cat /boot/cpu_manufacturer-ucode.img /boot/initramfs-linux.img > /tmp/combined_initrd.img
$ objcopy \
    --add-section .osrel="/usr/lib/os-release" --change-section-vma .osrel=0x20000 \
    --add-section .cmdline="/etc/kernel/cmdline" --change-section-vma .cmdline=0x30000 \
    --add-section .splash="/usr/share/systemd/bootctl/splash-arch.bmp" --change-section-vma .splash=0x40000 \
    --add-section .linux="vmlinuz-file" --change-section-vma .linux=0x2000000 \
    --add-section .initrd="initrd-file" --change-section-vma .initrd=0x3000000 \
    "/usr/lib/systemd/boot/efi/linuxx64.efi.stub" "linux.efi"
$ cp linux.efi esp/EFI/Linux/
```

其实既然都手动了，不妨再手动一下，连systemd-boot这个bootloader都不要了，直接用efibootmgr去生成一个`.efi`文件，这样UEFI会直接读这个文件引导进linux， 大概像这样：

```bash
efibootmgr --create --disk /dev/nvme0n1p1 --loader /EFI/Linux/archlinux_x64.efi --label "Archliux Boot" --verbose
```

mkinitcpio大概像这样，和用grub的时候差不多， 见： [ArchWiki Unified Kernel Image](https://wiki.archlinux.org/title/Unified_kernel_image#mkinitpcio)  

```bash
mkinitcpio -p linux -- --uefi esp/EFI/Linux/test-systemd.efi
```

systemd-boot会自动识别`boot/EFI/Linux`下面的unified kernel image文件。

### systemd-boot如何设置内核参数（参数是怎么传递的）

在options选项里面附加就可以了，然后systemd-boot会把参数传给initramfs，然后就是[kernel’s command-line parameters](https://www.kernel.org/doc/html/v4.14/admin-guide/kernel-parameters.html)那一套(能用`cat /proc/cmdline`看到的)，就是嵌入式uboot启动传参的那个tag链表。然后init的时候将参数传给systemd（既然你都用systemd-boot了那你的init系统肯定是systemd吧）

我以前看名字以为systemd直接就接管了，传给systemd之后就是熟悉的sysctl那一套`systemd-sysctl`。后来发现systemd-boot就是一个`bootctl`那种efi引导文件生成的一个工具（可以理解为一个bootloader差不多的东西），bootloader的界面那个时候就跟其他的bootloader差不多， 那个时候还没systemd什么事呢。

如果你用mkinitcpio和unified kernel image， 那就`cp /proc/cmdline /etc/kernel/cmdline`， 改一下这个配置文件，然后再生成一遍initramfs的img，initramfs挂载根文件的时候会把参数传过去

### 怎么启动时指定kernel parameters

在systemd-boot的开机选择画面按e

### systemd-boot开机怎么进rootfs

一开始用了比较扭曲的办法，后来发现这个只要在将UUID指定成boot分区的UUID就可以了

### systemd-boot开机怎么进bytfs snapshot

这个貌似还真没啥好办法，要用这个还是装回grub吧。如果想开机进snapshot,需要手动将那个rootflag指定成snapshot所在的subvol。reddit上有一种方案是同时保留systemd-boot和grub, 然后在BIOS里面把systemd-boot的启动项放在前面，但是我不想要grub了。

>To use a subvolume as the root mountpoint specify the subvolume via a kernel parameter using rootflags=subvol=/path/to/subvolume. Edit the root mountpoint in /etc/fstab and specify the mount option subvol=. Alternatively the subvolume can be specified with its id, rootflags=subvolid=objectid as kernel parameter and subvolid=objectid as mount option in /etc/fstab. It is preferable to mount using subvol=/path/to/subvolume, rather than the subvolid, as the subvolid may change when restoring #Snapshots, requiring a change of mount configuration, or else the system will not boot.

如果要用systemd-boot进snapshot，需要改systemd-boot的配置文件指定rootflags。既然要手动改，不如直接用livecd进chroot操作更方便一点。需要手动改systemd-boot的参数不如grub那样装上`grub-btrfs`之后用上下键选择用哪个内核或者进入snapshot进行修复。

### 重启

可以在systemctl命令中直接选择entry

```bash
systemctl reboot --boot-loader-entry=auto-windows
```

## 其他的一些操作

### 如何加快启动速度

换成systemd-boot已经很快了，不过如果你想再快，可以用`systemd-analyze`, 看一下是什么拖慢了你的启动速度

```bash
$systemd-analyze
Startup finished in 11.282s (firmware) + 4.842s (loader) + 3.317s (kernel) + 7.955s (userspace) = 27.398s
graphical.target reached after 7.955s in userspace
```

如果想更详细的调查一下， `systemd-analyze blame`或者`systemd-analyze plot > boot_process.svg`

```bash
$systemd-analyze blame
4.841s NetworkManager-wait-online.service
2.429s systemd-udev-settle.service
2.144s systemd-modules-load.service
 579ms systemd-binfmt.service
 496ms systemd-timesyncd.service
 320ms dev-nvme0n1p1.device
 223ms docker.service
 157ms systemd-journal-flush.service
 129ms systemd-remount-fs.service
  93ms udisks2.service
  84ms boot.mount
  83ms user@1000.service
  64ms libvirtd.service
  63ms lvm2-monitor.service
  38ms systemd-udev-trigger.service
  37ms systemd-journald.service
  36ms upower.service
  33ms systemd-tmpfiles-setup.service
  32ms systemd-udevd.service
  30ms systemd-tmpfiles-clean.service
  30ms systemd-logind.service
```

可以看到是NetworkManger拖慢了我们的速度，然后发现是`systemd-networkd-wait-online.service`, 如果我们不需要这个直接禁用就好了

一般systemd-boot和systemd开机那一堆service和DM和DE（Desktop Environment）的加载相比不会拖慢你的速度的。不过我们其实可以连bootlaoder都不要的，如果是UEFI， 那直接EFI Stub就可以了

BIOS加载bootloader的启动文件（比如systemd-boot的systemd-bootx64.efi），bootloader再去引导内核启动

```c
❯ tree EFI
EFI
├── BOOT
│   └── BOOTX64.EFI
├── Linux
└── systemd
    └── systemd-bootx64.efi
```

其实我们可以直接用efibootmgr生成linux的efi启动文件，然后跳过booloader这个中间商。同样指定ucode和initramfs的img， 以及root所在的分区，然后会生成一个efi文件，直接引导进linux

### 直接跑在内存上

Archlinux可以直接用ramroot, 这个只要装个AUR包然后配置一下就可以了

## 说明

### 关于UEFI和BIOS

UEFI确切的说法是UEFI BIOS，其实也算作BIOS的一种，通常所说的BIOS指Legacy BIOS。这里按照通常习惯简称UEFI BIOS为UEFI, Legacy BIOS为BIOS。

### bootloader

linux在很早的时候是不需要bootloader的，后来有了bootloader这种东西，不过你还是可以不要bootloader的，比如上面说到的EFI Stub。
systemd-boot也算是个bootloader，不过相比其他的booloader已经精简了很多了。

### 关于initramfs和rootfs

看这里： <https://www.kernel.org/doc/Documentation/filesystems/ramfs-rootfs-initramfs.txt>  
<https://wiki.gentoo.org/wiki/Initramfs/Guide/zh-cn>  

### 傻多戴的阴间BIOS固件

你戴的阴间BIOS固件，强制升级不能回退就算了，每次TM升级bug不修还锁CPU频率，次次更新BIOS反向升级，想淘汰老用户就直说呗。当年我想不要grub了整个EFI Stub玩，结果发现你戴的阴间BIOS怎么还是不行。。。

>EFISTUB does not work on some Dell systems
>Several generation Dell firmwares are wrongly passing the arguments to the bootloader, thus making EFISTUB parse a null command line which normally means unbootable system (see the complete linux-efi thread).

>A workaround has been found since Linux 5.10 to correct this behavior (see this commit ). For Linux < 5.10, you can use an efi-packer like arch-efiboot, or a different bootloader.

从13年到现在用过3台dell的笔记本了，一个比一个糟心，我再买dell的笔记本我是XX

## 链接

[Arch boot process(简体中文)](https://wiki.archlinux.org/title/Arch_boot_process_(简体中文))  
[浅析安全启动（Secure Boot）](https://bbs.pediy.com/thread-260399.htm)  

[Writing an x86 "Hello world" bootloader with assembly](http://50linesofco.de/post/2018-02-28-writing-an-x86-hello-world-bootloader-with-assembly)  
[systemd时代的开机启动流程(UEFI+systemd)](https://www.junmajinlong.com/linux/systemd/systemd_bootup/)  
[Linux内核参数传递Tag(init_tags)](http://blog.chinaunix.net/uid-9543173-id-1989004.html)  

[systemd The Boot Loader Specification](https://systemd.io/BOOT_LOADER_SPECIFICATION/)  官方文档  

[init (简体中文)](https://wiki.archlinux.org/title/Init_(简体中文))
<https://wiki.archlinux.org/title/Arch_boot_process>  
<https://wiki.archlinux.org/title/Systemd-boot>  
<https://wiki.archlinux.org/title/EFI_system_partition>  
<https://wiki.archlinux.org/title/Unified_kernel_image>  
<https://wiki.archlinux.org/title/microcode>  
<https://wiki.archlinux.org/title/EFISTUB>  

[Multi Boot Linux With One Boot Partition](https://ramsdenj.com/2016/04/15/multi-boot-linux-with-one-boot-partition.html)  
<https://en.wikipedia.org/wiki/List_of_Linux_distributions_that_run_from_RAM>  
<https://github.com/arcmags/ramroot>  
<https://www.kernel.org/doc/html/v4.14/admin-guide/kernel-parameters.html>  
<https://www.kernel.org/doc/Documentation/efi-stub.txt>  

<https://wiki.gentoo.org/wiki/Genkernel> 隔壁gentoo的genkernel，感觉比Arch的mkinitcpio定制性高不少  
[systemd-boot 中文手册](http://www.jinbuguo.com/systemd/systemd-boot.html)  

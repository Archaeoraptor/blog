---
title: Linux系统的抢救和维护
date: 2021-04-09 20:31:56
tags:
- Linux
- rescue
abbrlink: 'linux-rescue'
---
这篇水文主要是写给桌面玩家看的，如果您是运维，上线的服务器打出了GG/被黑了/库被删了，与其来这里找答案，不如赶紧去看看适合删库跑路人士的班次，连夜扛着火车跑路吧
<!-- more -->

## 通常的卡死解决流程

（此处的“卡死”，指鼠标动不了或某个应用、进程没有响应了，以下流程展示以KDE和Archlinux为例）

卡死可能有很多情况，可能内存泄漏或应用太多把内存和缓存沾满了（比如Ubuntu16的GNOME桌面祖传内存泄漏），可能是某些Wine应用，可能是某些应用对某些桌面发行版兼容性不好（比如linuxqq），可能是显卡驱动或混成器的问题。有的时候，什么都不干都会发生卡死，比如GNOME3和KDE4。

现在的Linux桌面和软件已经足够日常使用了，但是日常使用还是会发生卡死的现象，比如vscode内存泄漏，比如baloo，比如pipewire和pulseaudio。

相比于Linux/Unix在服务器上

首先尝试`ctl+alt+t`呼出终端，如果能，那问题应该不大，然后用`htop`之类的东西看一下罪魁祸首是谁

如果是KDE桌面及其组件的问题

`plasmashell --replace`

如果不能就`ctl+alt+F2`调出tty，htop看一下那个进程占用高，kill掉，如果不行就重启tty

```bash
kquitapp5 plasmashell && kstart5 plasmashell
```

正常退出不行就试试killall

```bash
killall plasmashell && kstart5 plasmashell
```

或者重启一下SDDM

```bash
systemctl restart sddm
```

还不行就杀掉tty1，然后重启xorg

```bash
ps aux | grep tty1 # 或者pgrep tty1
```

然后kill掉（或者直接`pkill -9 -t tty1`），重启。或者这样

```bash
pkill x
startx
```

那如果你连tty都切不过去，就用魔术键重启。（如果你提前设置开启了魔术键SysRq的话，这个下面再讲）
按住`Alt+Shift+SysRq`，依次按`REISUB`这几个键。（每按一个键停几秒钟）（如果你内核出了kernel panic之类的问题，那魔术键也没用）

再不行直接`reboot`，`shutdown -r now`就好了

以上的命令如果桌面经常卡死建议绑定到你喜爱的快捷键上（很多时候桌面卡住了，然后）

再不行就直接尝试长按电源关机了（一般长按电源键5-10秒，这取决于你的主板BIOS设置，这会发送一个RESET信号）。如果你的主板有复位键，就按复位键重启。实在实在没办法那就拔电源吧。（做好丢失所有未保存数据的准备）

ps: 如果你用的ext4，那恭喜你，中奖了。不是丢失数据这么简单了，你的文件系统可能已经出了小问题。（这个后面再说）


## 碰到比较严重情况的抢救

这种情况包括但不限于不小心卸载了系统自带的python、卸载了iptables、卸载了桌面环境（DE）或者卸载了什么重要的库，或者你改了什么`kernel parameter`直接黑屏进不去了，又或者你执行了传说中的`rm -rf /`

有或者显卡驱动有问题了

或者双系统Windows更新把你grub整没了（不要双系统就没这么多事，直接上`systemd-boot`之类的东西）

总之，开机直接黑屏了。或者你运气好，能进grub界面，还能抢救一下。或者运气更好，进了grub界面然后在DM那里黑屏了（这种可能是显卡驱动的问题）

你可能需要一个LiveUSB用来抢救。如果你没设置加密，那就试试能不能挂载原来的硬盘。

```bash
sudo pacman -S base base-devel linux linux-firmware linux-headers xorg plasma kde-applications
```

### 文件系统损坏

比如，ext4断电

如果你的电脑要经常面对断电等恶劣情况，那建议去用ZFS等文件系统，不要用ext4

### kernel panic 等

kernel panic segmentation fault core dump

https://wiki.archlinux.org/index.php/Core_dump#Examining_a_core_dump

https://wiki.archlinux.org/index.php/General_troubleshooting

### 看看是不是硬件坏了

内存条没插紧、硬盘坏了。。。。

## 排查问题

排查问题和调试是不太好讲的问题
![](linux-rescue/1618031552.png)


kdump crash kernel serial console netconsole

## 备份

>备份，永远的神

推荐使用rsync或timeshift（）
btrfs用户可以使用btrfs snapshots

## 日常维护

### 测试

一些可能会带来的操作尽量现在虚拟机、容器里面试一下

### dotfiles和脚本

经常换电脑或重装可以准备dotfiles和快速安装脚本

### 安全问题

大部分Linux桌面玩家应该没啥可担心的安全问题，自用台式机、笔记本如果没有暴露在外的公网IP和端口，应该不用担心。（毕竟桌面玩家大部分连个独显都没有，挖矿都榨不出油水）


## 一点不成熟的建议和个人体验

混成器（compositer）有OpenGL2、OpenGL3.1、xrender三个选项，混成器和显卡驱动设置不当经常会出一些小问题，比如Kwin屏幕撕裂、黑屏、卡死。如果你的鼠标卡住动不了了，可以在重启plasma之前禁用混成试试（默认的快捷键是Alt+Shift+F12）

![](linux-rescue/1618028977.png)

对于混成器感兴趣可以看看fc老师的这篇博客：[桌面系统的混成器简史](https://farseerfc.me/zhs/brief-history-of-compositors-in-desktop-os.html)，不过现在2021年了，wayland还是。。。

如果你用Nivida，或者KDE，或者除了gdm以外的其他dm,不太推荐wayland。如果你喜欢触控板和GNOME,可以试试。

KDE用户可以考虑关闭baloo（这是一个搜索工具，然而经常出现内存占用飙升、CPU 100%）

> 如果你使用 [qt5ct](https://archlinux.org/packages/?name=qt5ct) 包，Qt5 设置工具将有可能覆盖系用设置内的字体设置。

小白用户可能还是适合Windows，如果用Linux就老老实实默认设置，不要为了好看去照着不知道哪来的教程做一些自己也不知道干了什么的美化。

如果真想入坑定制和美化，不妨去reddit的 Unix Porn 看看。

休眠设置不当经常导致很多问题，swap和zram要仔细配置

显卡驱动是万恶之源，Nvidia Fuck you！

使用wayland基本就是灵车漂移

使用pipewire也是

btrfs曾经是灵车，现在不是生产环境个人使用感觉还行（请善用snapshots备份）

虽然只用过几个月的Manjaro，可是

Linux用户包括Linux做主力的桌面玩家，靠这个吃饭的服务器运维或者网管，驱动和嵌入式开发们、红帽和Canonical

更多的人只是偶尔连上去服务器的或者有一台不常用的桌面系统（大概率是Ubuntu，还是跑在虚拟机里的那种）

https://thomask.sdf.org/blog/2019/11/09/take-care-editing-bash-scripts.html

https://news.ycombinator.com/item?id=23087308

https://juejin.cn/post/6844904122609565703

YuutaW SIGFIG, [10.04.21 09:50]
[In reply to YuutaW SIGFIG]
按照 archwiki 加了 single irqpoll maxcpus=1 reset_devices cmdline 就是这样（

YuutaW SIGFIG, [10.04.21 09:50]
不加的话 panic 直接重启（BIOS 那边重启

Jerry Xiao, [10.04.21 09:51]
[In reply to YuutaW SIGFIG]
reuse旧的就可以

Jerry Xiao, [10.04.21 09:51]
或者就加一个 reset_devices?

Jerry Xiao, [10.04.21 09:52]
还可以加一个 cmdline 让 systemd 启动 emergency.target


echo b > /proc/sysrq-trigger

https://wiki.archlinux.org/index.php/Kdump
http://www.brendangregg.com/blog/2019-07-15/bpf-performance-tools-book.html
http://www.brendangregg.com/linuxperf.html
https://en.wikipedia.org/wiki/Magic_SysRq_key
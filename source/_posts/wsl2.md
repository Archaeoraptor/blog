---
title: 更新至wsl2.0.0，开启mirrored网络模式
date: 2023-10-23 09:21:08
tags:
- linux
- wsl
abbrlink:
- 'wsl2'
---
上个月wsl2发布了2.0.0，新特性很多，比如networkingMode新增了mirrored，可以像wsl1那样和宿主机共享网络（mirrored模式），也可以直接吃到windows的系统代理设置。
<!-- more -->

迫于生计，要用win10干活，只好在wsl里面跑几个linux玩一玩了。目前这个功能只有Windows Insider Preview Canary有，所以需要冒进到Canary channel才行。**注意，Canary是比dev和beta channel更新的预览通道，请谨慎更新。**

上个月wsl2发布了2.0.0，现在还是pre-release，所以需要手动更新。（建议开全局代理，不然下载更新很慢）

```powershell
wsl --update --pre-release
```

目前wsl2.0.0的新特性需要在`%userprofile%\.wslconfig`配置里面手动设置。

```config
[experimental]
autoMemoryReclaim=gradual
networkingMode=mirrored
dnsTunneling=true
firewall=true
autoProxy=true
sparseVhd=true
```

然后就可以用mirrored模式了，就像wsl1那样，宿主机和wsl网络互通，可以直接`127.0.0.1`去访问wsl中的端口（ipv6也可以），不需要手动端口转发了。

而且可以吃到Windows的代理设置，也可以用Windows的防火墙规则去过滤包。

新版wsl还有很多新特性，比如：

DNS Tunneling: DNS隧道。之前wsl2的DNS经常出问题，得在`/etc/wsl.conf`里面设置`generateResolvConf = false`，然后手动在`/etc/reslov.conf`里面指定DNS Server。如果你不改，那wsl的DNS会默认去找Windows本机的DNS，然后如果你随便配了点什么防火墙规则啊、有点什么VPN啊，或者又是整了个万恶的Docker，恭喜你，wsl尝试去找Windows的DNS的包发过去又失败了，你上不了网了。

>This is because the networking packet for DNS sent by the WSL VM to the Windows host was being blocked by the existing networking configuration.

这玩意还是偶发性的，因为wsl每次启动会随机, 哪次wsl启动的时候随机分配的子网，然后跟抽奖一样，哪次分配的子网就被防火墙给拦了，下次又好了。

这也是从wsl1就开始的祖传老bug了，我们看一下相关问题的赞数就知道这破玩意有多恼火： 
<https://gist.github.com/coltenkrauter/608cfe02319ce60facd76373249b8ca6>  
<https://stackoverflow.com/questions/62314789/no-internet-connection-on-wsl-ubuntu-windows-subsystem-for-linux/64057835>  
我是不明白为什么巨硬这么多问题还是执着的想让wsl的默认DNS用Windos宿主机一样的，直接像虚拟机一样用自己的不好吗。。。这个bug摆在这里这么久，最后巨硬决定整个DNS Tunneling

autoMemoryReclaim：自动回收内存，一定程度上缓解了我的16G内存的寒酸笔记本。但是这个也和docker有点小冲突，又是万恶的cgroupv1（docker啥时候才能支持cgroupv2啊，我等的花都谢了）。

这个选项有三个参数：gradual，dropcache和disable，gradual在空闲五分钟后一点一点缓慢释放内存（聪明的你应该猜到了，又是用cgroup实现的），dropcache会直接释放所有空闲内存。

sparseVhd: 自动释放虚拟硬盘，没啥好说的。

wsl2 2.0.0新版还是没有解决和windows互传文件性能拉跨的问题。建议继续使用9p virtfs（就是qemu用的那个）。

## 链接

[Windows Subsystem for Linux September 2023 update](https://devblogs.microsoft.com/commandline/windows-subsystem-for-linux-september-2023-update/)  
[Advanced settings configuration in WSL](https://learn.microsoft.com/en-us/windows/wsl/wsl-config)  
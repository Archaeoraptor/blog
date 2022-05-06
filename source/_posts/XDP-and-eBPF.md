---
title: XDP、eBPF和内核旁路：网络的新玩意
date: 2022-04-28 19:44:06
tags:
- network
- XDP
- eBPF
abbrlink: 'XDP-and-eBPF'
---
之前弄软路由和透明代理的时候，linux下的方案几乎底层全是iptables的，就转发个。后来看到kernel offload之类的东西，再后来又看到XDP, 一开始没在意，后来突然想起这不就是......
<!-- more -->

有一些专用网卡，会用硬件接管一些offload的工作，比如LSO、TSO啥的，这里不谈这些。

## 内核旁路（Kernel Bypass）

之前在看xv6和linux的协议栈，那一阵正好没事也看看了网络虚拟化玩，产生了一个比较困惑的问题：

好多东西都为了性能挪到用户态去，为什么不把网络协议栈做到用户态去？

然后看到dxdk就是在用户态搞，还有隔壁BSD的netmap。（DXPK这个玩意我不建议没有相当折腾能力的去玩，基本上要手搓用户态协议栈，然后周边配套工具也没有）

## eBPF

一开始知道eBPF是从bpftrace那听说的，后来才发现原来tcpdump就是基于eBPF的（libpcap），所以能在网卡入口抓，在iptables丢包之前抓到网卡过来的包。

eBPF的前身BPF在BSD那是很早就有的东西了，

## XDP

```bash
yay -S xdp-tools
```

## 链接

[xdpcap: XDP Packet Capture](https://blog.cloudflare.com/xdpcap/)  
[A brief introduction to XDP and eBPF](https://blogs.igalia.com/dpino/2019/01/07/a-brief-introduction-to-xdp-and-ebpf/)  

[如何每秒丢弃1000万个数据包](https://blog.cloudflare.com/zh-cn/how-to-drop-10-million-packets-zh-cn/)  
[使用 XDP-FILTER 进行高性能流量过滤以防止 DDOS 攻击](https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/8/html/configuring_and_managing_networking/using-xdp-filter-for-high-performance-traffic-filtering-to-prevent-ddos-attacks_configuring-and-managing-networking)  
[]()  
[[译] Cilium：BPF 和 XDP 参考指南（2021）](https://arthurchiao.art/blog/cilium-bpf-xdp-reference-guide-zh/)    
[[译] Facebook 流量路由最佳实践：从公网入口到内网业务的全路径 XDP/BPF 基础设施（LPC, 2021）](http://arthurchiao.art/blog/facebook-from-xdp-to-socket-zh/)  
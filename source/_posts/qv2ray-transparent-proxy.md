---
layout: posts
title: 『转载』使用Qv2ray+cgproxy配置透明代理（仅限Linux）
date: 2020-10-29 19:46:13
hide: true
abbrlink: 'qv2ray-transparent-proxy'
tags:
 - qv2ray
 - cgproxy
 - transparent-proxy
---

从Qv2ray电报群里转来的，透明代理老有人问，感觉这个方案比tproxy等方案好，转出来。版权归Qv2ray及cgproxy所有

<!-- more -->

用的是这个项目：<https://github.com/springzfx/cgproxy>

## 透明代理设置

1. 在“首选项-入站设置”的下方启用透明代理选项。

   - 监听ipv4地址可填127.0.0.1或0.0.0.0，建议前者。若需双栈代理，则在监听ipv6地址填上`::1`（如果监听ipv4填了0.0.0.0则可不填）。
   - 在“网络模式”中勾选需要透明代理的协议。模式选择“tproxy”。
   - 如果希望在透明代理环境里让v2ray的内置dns接管本地dns，则勾选“dns拦截”。注意，在透明代理环境下，如果系统dns或v2ray的内置dns配置不当，可能导致系统无法解析域名从而无法正常上网。详见后文说明。

   如果是复杂配置，则需要手动添加相应的dokodemo-door入站。由于目前版本复杂配置并没有提供tproxy选项，因此tproxy模式需要通过编辑json来实现。

2. 安装`cgproxy`软件

   - `cgproxy`软件已在archlinux, fedora 32, ubuntu 18.04, ubuntu 20.04, deepin 15.11, deepin v20 beta发行版中测试过。
   - Archlinux用户可直接在AUR上安装，deb或rpm系发行版用户可从[github](https://github.com/springzfx/cgproxy/releases)上下载安装包。非以上发行版用户，可自行从[github](https://github.com/springzfx/cgproxy)上获取代码自行编译。

3. 配置`cgproxy`，编辑`/etc/cgproxy/config.json`：

   - 在`cgroup_proxy`中括号里加上"/"（包含引号），`port`改为Qv2ray首选项里的透明代理的端口。
   - `cgproxy`默认配置是代理所有tcp和udp，ipv4和ipv6的流量，如果不希望代理其中的某种（些）流量，则将对应的`enable_xxx`改为false。注意这里的配置要和Qv2ray选项里的配置一致（如，Qv2ray选项里没有勾选udp，则这里务必把`enable_udp`改为false）。
   - 如果希望当本机作为网关设备时为连接到本机网关的其他设备（如连接到本机开设的wifi热点的设备）也提供透明代理，则把`enable_gateway`改为true。

4. （重要）透明代理的基本原理是拦截系统发出的所有流量，并将这些流量转到代理工具里，从而实现让系统所有流量都走代理的目的。此时，为了避免流量出现死循环（即代理工具发出的流量又转回到代理工具里），需要将代理工具排除在透明代理环境外面。有两种方式可以实现这一点：

   - 通过`execsnoop`监控代理工具的启动，并自动将其移至透明代理环境外面：

     - `cgproxy`软件自带`execsnoop`支持，以上`cgproxy`测试过的发行版均可支持。
     - 编辑`/etc/cgproxy/config.json`，在`program_noproxy`中括号里加上"v2ray","qv2ray"（包含引号和逗号），以使`qv2ray`和`v2ray`发出的流量不经过透明代理。如果你的`v2ray`或`qv2ray`不在`PATH`里，则需要填写它们的绝对路径。

   - 在每次连接代理节点时，让`qv2ray`自己把自己移到透明代理环境外面：

     - 安装Qvplugin-Command插件，在插件设置里的“pre-connection”栏里加上一句

       ```bash
       sh -c "cgnoproxy --pid $(pgrep -x qv2ray)"
       ```

       即可。

5. （重要）如果启用了udp的透明代理（dns也是udp），则给v2ray二进制文件加上相应的特权：

   ```bash
   sudo setcap "cap_net_admin,cap_net_bind_service=ep" /path/of/your/v2ray
   ```

   否则udp的透明代理可能会出问题。

6. 启动透明代理服务：`systemctl start cgproxy.service`或`systemctl enable --now cgproxy.service`。

以上步骤完成后，透明代理应该能正常使用了。

## dns配置说明

如果勾选了“dns拦截”，且启用了dns和udp的透明代理，则v2ray会拦截对系统dns的请求，并将其转发到v2ray的内置dns里，即让v2ray内置dns接管系统dns。但v2ray内置dns是会遵循路由规则的。

如果没勾选“dns拦截”，则v2ray虽然不会让内置dns接管系统dns，但如果启用了dns和udp的透明代理，则系统dns也会走透明代理进v2ray，并遵循v2ray的路由规则。

因此，在启用了dns和udp的透明代理时，若系统dns或v2ray的内置dns配置不当，可能导致dns请求发不出去，从而影响正常上网。

由于qv2ray默认的路由规则是绕过国内ip，国外ip均走代理。在这个情形中，以下两个配置是典型的有问题的dns配置方式：

- 配置了国外普通dns作为首选，但代理本身不支持udp（此时dns查询的udp流量出不去，dns无法查询）
- 配置了使用域名的doh作为首选（此时doh的域名无法解析，从而doh也无法使用）

一般而言，如果并不在意将dns查询发给谁，那么，在绕过国内ip的情况下，只需要配置一个国内普通dns作为首选即可保证不会出问题。若代理本身不支持udp，又希望使用国外dns，则可以考虑使用使用ip的doh（如`https://1.1.1.1/dns-query`等）。

如果需要更复杂的dns配置，建议参考[上游文档](https://www.v2ray.com/chapter_02/04_dns.html)，并选择合适的不会影响正常上网的dns配置。

## 常见问题

- 启用透明代理后无法访问任何外网，且v2ray的cpu占用率飙升

  可能是流量陷入死循环了，检查第4步有没有正确配置。如果配置没问题，执行`systemctl status cgproxy.service`看下有没有诸如`info: process noproxy pid msg: xxx`之类的输出。如果没有，则说明cgproxy软件或execsnoop没有正常工作。注意cgproxy软件需要cgroup v2。

  尝试退出qv2ray，随后在终端里执行`cgnoproxy qv2ray`看是否恢复正常，如恢复正常，说明cgproxy正常工作，只是execsnoop没有正常工作。由于execsnoop一定程度上依赖于内核，非上述cgproxy测试过的发行版用户，建议使用第4步中的第2种方法。另外，对kde用户，5.19+版的plasma会给从krunner里启动的程序额外设置cgroup，尽管cgproxy软件考虑到了这一点，但仍有极少数场合可能出现plasma设置的cgroup覆盖掉了cgproxy设置的cgroup的情况，此时通常重启一下qv2ray即可。

- 启用透明代理后，无法访问（部分）域名

  可能是dns无法解析（部分）域名。一般这种故障只发生在启用了dns及udp透明代理的时候。

  终端里执行`dig 无法访问的域名`看下报什么错：

  - 若出现类似`reply from unexpected source: 192.168.0.100#42050, expected 8.8.8.8#53`的报错，则检查第5步的有没有正确配置。

  - 若出现类似`connection timed out; no servers could be reache`的报错，则说明dns查询的流量出不去，此时往往是系统dns或v2ray内置dns配置不当。请检查是否出现了前文提到的几种不当配置。如果没有勾选“dns拦截”，则此时v2ray虽然不会用内置dns接管系统dns，但它仍然会让系统dns走透明代理，从而遵循v2ray的路由规则，此时需要检查系统dns是否是前文提到的那几种不当配置。

- 能不能分应用代理（如，下载BT时不能走代理）

  对于本机的程序，可以，可通过两种方式实现：

  - 通过`cgnoproxy`实现：如，在命令行中执行`cgnoproxy qbittorrent`，启动的qbittorrent程序就不会走透明代理。又如，在命令行中执行`cgnoproxy --pid 12345`，执行之后pid为12345的程序就不再走透明代理。这种方式可支持任何应用。
  - 通过`/etc/cgproxy/config.json`实现：在配置里的`program_noproxy`中括号里加上相应的应用即可。这种方式只支持可执行文件，不支持各种脚本。注意修改`config.json`之后，需要重启cgproxy服务才能生效，执行`systemctl restart cgproxy.service`即可。

  对于当本机作为网关设备时为连接到本机网关的其他设备，不行，那些设备的所有流量（到本机的流量除外）都必然会走代理。

- 透明代理环境中响应速度变慢

  由于iptables是在域名解析成ip之后，才对相应的流量进行重定向。因此，在透明代理环境中，访问一个域名s可能会需要解析至少2次dns（系统解析一次，重定向到v2ray之后v2ray分流模块再解析一次）。因此，响应理论上是会变慢一点的，变慢的幅度取决于系统dns及v2ray的dns的响应速度。

## 自己写的更新和群友遇到的一些问题

### 排查问题

```bash
systemd-cgls /noproxy.slice # 检查一下被排除代理的应用
cgproxy curl -sSLv https://www.google.com/ # 开着代理连一下谷歌试试，检查一下你的透明代理是不是好的
cgnoproxy firefox # 临时关掉透明代理运行某个应用（比如Firefox），检查一下是不是透明代理造成的问题
```

### Docker和透明代理冲突的问题

这也是一个频繁被问到的问题

>你是装了某些可能会破坏 cgroup matching 的东西吗
>比如 docker 之类的肮脏程序
>docker 不仅会破坏cgroup matching
>把网络搞炸
>docker 还有 hairpin nat
>巨坑


我自己试了一下是这样的

在`systemctl enable docker`后没有问题

这个时候我没把当前用户添加到个人用户组，不能直接使用Docker，需要`sudo docker ...`
在我把Docker添加到当前用户组后出问题了，重启后qv2ray和clash等失效了，延迟测试全部显示0ms

报错

```bash
2020/11/19 20:34:03 192.168.1.105:58398 accepted tcp:211.72.35.152:80 [outBound_PROXY] 

2020/11/19 20:34:10 [Warning] [4118491953] v2ray.com/core/app/proxyman/outbound: failed to process outbound traffic > v2ray.com/core/proxy/vmess/outbound: failed to find an available destination > v2ray.com/core/common/retry: [v2ray.com/core/transport/internet/websocket: failed to dial WebSocket > v2ray.com/core/transport/internet/websocket: failed to dial to (ws://feec8af.rf.cloudflare.systems/s/feec8af.fm.apple.com:29306):  > read tcp 192.168.1.105:58370->211.72.35.152:80: i/o timeout v2ray.com/core/transport/internet/websocket: failed to dial WebSocket > v2ray.com/core/transport/internet/websocket: failed to dial to (ws://feec8af.rf.cloudflare.systems/s/feec8af.fm.apple.com:29306):  > dial tcp: operation was canceled] > v2ray.com/core/common/retry: all retry attempts failed
2020/11/19 20:34:10 [Warning] v2ray.com/core/transport/internet/tcp: failed to accepted raw connections > accept tcp 127.0.0.1:12345: accept4: too many open files
2020/11/19 20:34:10 192.168.1.105:58946 accepted tcp:211.72.35.152:80 [outBound_PROXY]
```

一个办法是用docker的时候加`sudo`以root用户运行。

另一个解决办法见：

<https://github.com/springzfx/cgproxy/issues/>

编辑`/etc/default/grub`，
添加`cgroup_no_v1=net_cls,net_prio` 到`GRUB_CMDLINE_LINUX_DEFAULT`中,
然后更新grub，重启
示例：

```bash
GRUB_CMDLINE_LINUX_DEFAULT="text cgroup_no_v1=net_cls,net_prio"
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

但是，加了这些参数经常也不太好用。

{% cq %}
最简单的办法，扬了 docker，换 podman / podman-docker
{% endcq %}

```bash
sudo pacman -Syu podman-docker
```

或者安装podman，然后

```bash
alias docker=podman
```

然后你需要编辑`/etc/subuid`和`/etc/subgid`加上`podman:165536:4096`，然后

```bash
usermod --add-subuids 165536-169631 --add-subgids 165536-169631 yourusername
```

不然会报错，没法pull images

```
ERRO[0000] cannot find UID/GID for user zjk: open /etc/subuid: no such file or directory - check rootless mode in man pages.
```

### pacman更新报错

本来都是好的，突然有一次更新问题了。浏览器等访问都没有问题，怀疑是透明代理的问题。Qv2ray输出看不到异常。

```log
:: Synchronizing package databases...
error: failed retrieving file 'core.db' from mirrors.uestc.cn : Failed to connect to mirrors.uestc.cn port 80: Connection timed out
error: failed to update core (download library error)
error: failed to synchronize all databases
error installing repo packages
```

使用`cgnoproxy yay -Syu` 看了一下，果然。
然后在cproxy中将pacman和yay加入noproxy组中。编辑`/etc/cgproxy/config.json`

```json
{
    "comment":"For usage, see https://github.com/springzfx/cgproxy",

    "port": 12345,
    "program_noproxy": ["v2ray", "qv2ray", "yay", "pacman"],
    "program_proxy": [],
    "cgroup_noproxy": ["/system.slice/v2ray.service"],
    "cgroup_proxy": ["/"],
    "enable_gateway": false,
    "enable_dns": false,
    "enable_udp": false,
    "enable_tcp": true,
    "enable_ipv4": true,
    "enable_ipv6": true,
    "table": 10007,
    "fwmark": 39283
}
```

### 透明代理和其他的代理设置冲突

比如SwitchyOmega、Firefox的代理设置

开了透明代理，理论上不需要对应用单独指定代理了，直接把SwitchyOmega关掉或者规则选 Direct，Firefox的代理设置也填不使用代理

### 报错 too many open files

报这种错:


>2021/03/11 22:32:32 [Warning] github.com/v2fly/v2ray-core/v4/transport/internet/tcp: failed to accepted raw connections > accept tcp 127.0.0.1:8889: accept4: too many open files


这个在开启UDP透明代理的时候常见, 可能是文件大小限制,也可能是你配置错误出现循环转发,比如[这个issue](https://github.com/v2ray/v2ray-core/issues/1563)

编辑`/usr/lib/systemd/system/v2ray.service` 或 `/etc/systemd/system/v2ray.service`

加入

```
[Service]
LimitNPROC=500
LimitNOFILE=1000000
```

然后: 

```
systemctl daemon-reload && systemctl restart v2ray
```

参考[v2ray配置指南的透明代理部分](https://toutyrater.github.io/app/tproxy.html#%E5%85%B6%E4%BB%96)

或者修改`/etc/security/limits.conf`在末尾添加

```
user   soft   nofile    40690
user   hard   nofile    40690
```

user改为你的用户名(`echo $USER`查看),或者想为所有用户设置就改为`*`（不推荐）
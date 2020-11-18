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


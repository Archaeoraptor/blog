---
title: VSCode使用ssh-key远程登录Ubuntu
tags:
  - ssh
  - vscode
categories:
  - 102网吧日常
abbrlink: 7f63
date: 2019-11-22 10:05:08
---
更新：已经把win10格了，跟Mobaxterm说再见了，Arch下vscode总是内存泄漏，感觉比win10下小问题也多不少

<!-- more -->

## 更新

### Linux下的一些问题

#### VSCode Remote远程登陆后打开终端报错

```bash
The terminal process failed to launch: Path to shell executable "/usr/bin/zsh" does not exist.
```

症状是这样的：我本地Arch的terminal是zsh，在VSCode的`erminal.integrated.shell`配置默认shell为zsh，远程连接的 Ubuntu Server 没装zsh、用的是非root用户登陆，登陆一切正常，然而呼出integrated terminal的时候就报错了。

然后我在issue评论区扒拉出来好多相同的问题，貌似是本地的VSCode配置会干扰远程

<https://github.com/microsoft/vscode-remote-release/issues/2579>
<https://github.com/microsoft/vscode-remote-release/issues/2313>

issue里面的解决方案有

>I have the same issue.Fixed after ssh as root

试了一下确实可行，不过root登陆是不可能root登陆的（这哪天又得被黑了）

还有：把VSCode里面的设置改为bash（不可能）

于是我给Ubuntu装了个zsh，好了。

#### VScode内存泄漏

我在Arch上不管是insider还是code-oss还是visual-studio-code-bin都有这个问题
（最后加钱把8G内存换成了32G，你随便泄漏去吧）

## 原文章

Mobaxterm挺好用的，只是那个文本编辑器不舒服。
试着用一下VSCode的remote-ssh
通过ssh-key登录就不多讲了[^1]

### Mobaxterm设置SSH密钥登录

ssh-keygen生成密钥之后在Mobaxterm的SSH设置找到Advanced SSH Setting里勾选import SSH key可以直接导入密钥。
退出再登录显示这样

```log
Authenticating with public key "Imported-Openssh-Key: id_rsa"
```

就是成功了（失败会提示这样`Server refused our key`)

### VSCode设置SSH密钥登录

装Remote-ssh插件，然后左边框会有个小电脑的图标，点那个图标，再点SSH TARGETS的设置（Configure），进行配置
它有个配置文件，一般在`C:/Users/<username>/.ssh`下面的config
编辑它的配置文件，Host随便填，HostName填ip，User填用户名，IdentityFile填私钥路径，Port填端口（默认22可以不填）
（要是认证失败什么的，建议把那些.ssh文件夹底下的文件删了，比如known_hosts）

```config
Host workstation
    HostName hostname
    User user
    IdentityFile C:/Users/XXX/.ssh/id_rsa
    Port 22
```

连接的时候可能会让你手动在termial里的信息
这样应该就行了, 不用每次输密码了。看到那个

什么？想用密码不想用ssh-key密钥登录，不能保存密码🐴？
您好，官方文档说不能
>Yes, you should be prompted to enter your token or password automatically. However, passwords are not saved, so using key based authentication is typically more convenient

（当然这里是新出的功能，bug当然少不了，我就遇到过[这样的](https://github.com/microsoft/vscode-remote-release/issues/1217), 一般遇到问题先升一下最新版或者用insider试试，再不行就去GitHub提issue）
其他的问题找官方文档[^2]看看，貌似可以SSH端口转发

远程SSH的VSCode插件是单独的，可以自行安装
如果你要配置多台主机SSH,在配置文件里并列加配置就行了

```config
Host workstation
    HostName hostname
    User user
Host Server
    HostName hostname
    User user
```

### SSH端口转发

老网民很熟悉的网上冲浪手段之一就是SSH Tunnel[^3]。当只开了22端口的时候可以用这种方法绕过防火墙用TCP之类的，当然，这种方法曾经也用来突破那个巨大的 防火墙 。
今天我突发奇想，看到工作站和服务器的专用线路的速度贼快啊，~~专用线路从来不低于45Mb/s不高于50Mb/s的速度毫不含糊~~，还有公网IP。
而102网吧由于老板太抠，四个人用一个路由器，辣鸡校园网网速10M/s都跑不到...

使用Mobaxterm和SwityOmega插件可以满足大部分需求

MobaXterm里选Tools > MobaSHHtunnel > New SHH tunnel > Dynamic Port forwarding (SOCKS proxy)

那个Dynamic Port 是动态转发，不是v2ray那种用来防止被封的动态端口，就是你本地的端口会被转到服务器上相同的端口，是sockets5代理
配置你的端口和密钥（最好把那个autostart和Autoconnect也选上），然后到SwityOmega之类的工具里面填上相应的端口，就可以愉快的上网了。从浏览器的IP查询网站看一下IP，已经是服务器的IP了

更新，看错了，忘了分Mbps和MBps了，实际上快不了多少，只有6MBps/s。但是没人跟我抢，自己走专线还是快一点的。

终端的命令可以看这篇，不多说了[^4]

### 题外话

新生代VPN, 底层貌似是UDP，红龙LOGO挺拉风的(跑题了
WireGuard据说要加到新的Linux Kernel里面，但是现在教研室的老Ubuntu只能自己装（这个我18年就听说了要加进kernel啦，连Linus都说好要加辣[^5]然而鸽到现在，还是没加，据说作者在搞windows的,现在做的差不多了，要回来折腾Linux了）
之前WireGuard由于流量特征太明显就换了v2，现在试试这个
安装比Openvpn简单，Ubuntu直接PPA装就行了（需要root权限）

```bash
sudo add-apt-repository ppa:wireguard/wireguard
sudo apt-get update
sudo apt-get install wireguard
```

win10的官方客户端已经出了，之前18年那会我还用的Tunsafe，设置参考这里[^6]
听八卦说Tunsafe和WireGuard作者还有过撕逼，此处不表

### 其他

#### 开机自启

```bash
systemctl enable wg-quick@wg0
```

#### 使用VSCode愉快的写C/C++

装C/C++和coderunner插件就行了，现在可以自动配置运行和调试的文件了（就那个以前一堆坑还经常改一更新就配置失效的launch.json和config.json）

#### VSCode连接Docker

安装Docker和Docker compose插件（在远程里面），然后直接用就行了 

## 注

[^1]:[设置 SSH 通过密钥登录](https://hyjk2000.github.io/2012/03/16/how-to-set-up-ssh-keys/)  
[^2]:[官方文档：Remote Development using SSH](https://code.visualstudio.com/docs/remote/ssh)  
[^3]:[墙与梯的较量](https://blog.yandere.moe/moe/gfw-vs-proxy/97.html)  
[^4]:[实战 SSH 端口转发](https://www.ibm.com/developerworks/cn/linux/l-cn-sshforward/index.html)  
[^5]:Linus在邮件中回复说：
>Btw, on an unrelated issue: I see that Jason actually made the pull
>request to have wireguard included in the kernel.
>
>Can I just once again state my love for it and hope it gets merged
>soon? Maybe the code isn't perfect, but I've skimmed it, and compared to the horrors that are OpenVPN and IPSec, it's a work of art.
见<https://lkml.org/lkml/2018/8/2/663>
[^6]:[Setting up WireGuard on Windows](https://golb.hplar.ch/2019/07/wireguard-windows.html)

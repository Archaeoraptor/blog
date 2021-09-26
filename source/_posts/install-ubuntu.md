---
title: 重装Ubuntu
tags:
  - ubuntu
  - 重装
categories:
  - 102网吧日常
abbrlink: 2d5
date: 2019-11-18 17:00:54
---

祖传服务器传到了我的手里, 行吧。其实我想把标题改成Ubuntu内部错误的解决办法（不是
<!-- more -->

## 更新（和洗手）

19年写的这个20年不知道怎么在2020年有一段时间被扔到了`重装Ubuntu`搜索结果的最前面，我这个小破站访问量暴增。然而那个时候20.04LTS都出了，已经有点过时了，而我19年那会写的那个几乎属于个人吐槽，差不多是我博客迁到，完全不适合作为一个教程。可能骗了很多人进来，

装什么Ubuntu啊，还是Arch香，群里个个都是人才，说话又好听

![](install-ubuntu/1611832700.webp)

但是祖传的是Ubuntu，但是假期我要回家滚又不敢远程滚, 但是我不想折腾显卡驱动了（Nvidia,fuck you!），但是这是工作站和服务器，上Arch天天滚可不太好。

~~但是随便装个Ubuntu不仅快，他们跑深度学习的时候抄CSDN配置环境也方便~~

~~最重要的，又不是我用。~~

~~GNOME内存泄漏关我什么事~~

~~Ubuntu内部错误关我什么事~~

还是去用Ubuntu吧.jpg

![](install-ubuntu/1611834396.webp)

![](install-ubuntu/1611834449.webp)

## 认真发言

其实你可以试试Debian（Canonical强推snap太烦人了）

### 关于桌面（DE）

带桌面的版本建议避开Ubuntu的GNOME，那个东西。。。

可以试试Linux Mint，或者Fedora（这个的GNOME应该比 Ubuntu 16、18 那几个整天内存泄露的GNOME版本稳多了）

其实我推荐KDE，可以试试Kubuntu（或者KDE Neon？）

机器配置较差请直接Lubuntu，稳如老狗，除了略简陋没有别的缺点。

一众WM用户应该不会搜索这么蠢的关键词搜到我这里来，这里不谈WM了。

### 关于服务器

小作坊没有专职运维和技术支持，其实服务器建议试试Debian或者Ubuntu LTS，Debian stable稳如老狗

~~众所周知，Debian testing 都比 Ubuntu LTS 稳（注意这个安全更新比较慢，个人用户可以testing，开了公网端口的工作站不要用这个）~~

不过Ubuntu和CentOS6、7比起来，还是瞬间和蔼可亲了不少

求求新入坑的别再跟着什么鸟哥私房菜之类的老教程用CentOS了，2020年CentOS8都被扬了，REHL马上就要给小作坊免费了，小作坊真迷信稳定上REHL吧

没事不要用所谓稳定的老版本然后找不到包只能自己手动编译，就不说c/c++那一堆lib了，我已经见过不少人在编译python的时候没开ssl当场翻车。

什么你说容器？你试过在CentOS6上面弄Docker和k8s吗，我在教研室的祖传阿里云上试过一次，从此再也不想看到CentOS了

（整天在CentOS上手动编译凸显运维重要性和工作量的除外）

### 太长不看

我是说，除了Ubuntu（以及其默认捆绑的GNOME、gtk、rime输入法,强推的snap，祖传的内部错误）,还有很多选择

或者你用WSL之类的东西也行呢（WSL2显卡驱动也差不多能用了，虽然io效率还有些问题）。

<!-- 上回书说到，服务器一不小心卸载了iptables
现在准备重装。还是打算装Ubuntu Server（并不打算装Manjaro，也不适合搞桌面版，今天试过一下Ubuntu Desktop，图形化界面由于服务器没有独显直接卡到爆炸）
~~初步打算安装Ubuntu server 16.04.6LTS, 使用LVM分区~~
安装Ubuntu18.04LTS，之前VPS用过一阵18，也没注意有什么新特性，据说还是改了一些东西⬇

>据说
内核更新到了 Linux kernel 4.15，这就意味着新增了不少特性 bbr什么的
网络管理工具 ifupdown 已经彻底废弃，建议全面拥抱 ip 命令
DNS 由 systemd 全面接管，接口监听在 127.0.0.53:53，配置文件在 /etc/systemd/resolved.conf，修改后重启服务即可 systemctl restart systemd-resolved。不要听网上那些奇怪的教程教你如何修改 /etc/resolv.conf
swap 文件来替代 swap 分区

## 装Ubuntu16.04LTS

### 安装

下载iso，制作启动盘，这里用的Rufus
![Picture](https://raw.githubusercontent.com/Archaeoraptor/image_resources/ImageofBlog/rufus.png)

然后直接装，网卡直接自动识别了，配置也没什么好说的，可以参考这里[^1]或这里[^2],

### 后续配置

进去之后换你电镜像源（终端里vim改/etc/apt/sources.list）
apt包update&&upgrade
修改时区和系统时间
安装Docker，照着官方教程装Docker的时候根本下载不动,换了个国内的源
桌面不打算装gnome或者kde了，Mobaxterm用X11一般就够了

## 重新安装Ubuntu18.04 LTS

由于某些原因，决定还是换18

## 再次安装

UEFI启动
create bond这里先不配置了（只有一张网卡eno1），bound mode选默认的balance-rr
eno1网卡选择automatic自动获取ipv4和ipv6
镜像源这里输入你电的
选use the entire disk and set up LVM
~~是否安装openssh，空格选安装，选择从GitHub获取~~
然后输入你的用户名(先准备好key，当然这个时候要保证网也是通的)
这里勾选openssh之后一直报错，可能是网的问题，不勾选，等后面再装
分区就分一个/boot，一个/swap，一个/home
<!-- 
（当然你要先准备好key，先用git bash生成key
`ssh-keygen -t rsa -C 'your_email@example.com'`
再到Github的SSH设置里面把公钥填进去（id_rsa.pub里面的内容）
然后测试一下是否成功
![Picture](https://raw.githubusercontent.com/Archaeoraptor/image_resources/ImageofBlog/ssh-github.png)
） -->

<!-- 然后是漫长的installing kernal
（不知道为什么装server比desktop版慢那么多）

bbr已经自带了，不用装了
设置root用户和用户组权限
装Docker参考这里[^3]
和英文官方教程那个差不多，主要是获取GPG的地址和Docker源换成国内的
换成国内中科大的镜像源（这个换了也时断时续可能连不上）
出现permission denied：

```log
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.40/containers/json: dial unix /var/run/docker.sock: connect: permission denied
```

是权限的问题，要加当前用户到用户组里面

```bash
sudo gpasswd -a ${USER} docker
```

### 更改静态为ip

网卡是eno1（服务器就一个网卡）
Ubuntu18的配置改了，在`/etc/netplan/50-cloud-init.yaml`, 现在18使用netplan管理网络
配置文件是yaml格式，跟hexo和next主题的配置文件格式一样

```yml
# network: {config: disabled}
network:
    ethernets:
        eno1:
            dhcp4: true
    version: 2
```

改成

```yml
network:
  version: 2
  renderer: networkd
  ethernets:
    eno1:   #配置的网卡名称
#      dhcp4: true    #dhcp4关闭
#      dhcp6: true    #dhcp6关闭
      addresses: [211.83.111.224/23]   #设置本机IP及掩码
      gateway4: 211.83.110.1   #设置网关
      nameservers:
          addresses: [114.114.114.114, 8.8.8.8]   #设置DNS
```

(网关地址用`route -n`查看，配置参考[ubuntu 18.04 netplan yaml配置固定IP地址](http://blog.sina.com.cn/s/blog_5373bcf40102xk5g.html))
d-homed\//g → /systemd-homed/
INFO  Path converted: /(?<=[(<s])(.\/)?systemd-homed\//g → /systemd-homed/
INFO  Path converted: /(?<=[(<s])(.\/)?systemd-home着默认
直接改成这样试试

```yml
network:
  version: 2
#  renderer: networkd
  ethernets:
    eno1:
      addresses: [211.83.111.224/23]
```

然后进/run/systemd/network看一下系统的网络配置文件

```bash
cd /run/systemd/network`
ls
cat *
```

下面这样应该就可以了，吧.....？

```log
root@xi102server:/run/systemd/network# cat 10-netplan-eno1.network
[Match]
Name=eno1

[Network]
LinkLocalAddressing=ipv6
Address=211.83.111.224/23
```

行吧，搜出一堆东西都不靠谱，还是得看官网

### 感想

服务器启动的声音真的大啊
一半以上的问题都是网络问题，校内这个网真的....
靠镜像源救回半条狗命

Ubuntu18安装时配置网卡时想选静态ip选择Manual进去填写subnet, 结果那个subnet选项是`XXX.XXX.XXX.XXX/XX`
看了教研室其他设备相同频段的子网掩码是`255.255.254.0`，但是填了不行，用网上计算子网掩码的工具算了一下，就是`255.255.254.0`

![Picture](https://raw.githubusercontent.com/Archaeoraptor/image_resources/ImageofBlog/netmmask.png)

暂时选了DHCP直接配置网卡，等进去再指定静态ip[^4]

哦，netplan好像把子网掩码的格式改了
原来是这样
>address = 192.168.225.50
netmask = 255.255.255.0

改成了这种格式
>addresses : [192.168.225.50/24]

滚去补习计网知识了 --> -->

## 安装参考

[tutorial-install-ubuntu-server](ttps://tutorials.ubuntu.com/tutorial/tutorial-install-ubuntu-server)  
 [LVM arch-wiki](https://wiki.archlinux.org/index.php/LVM_(简体中文))  
[Ubuntu 安装 Docker CE](https://yeasy.gitbooks.io/docker_practice/install/ubuntu.html)、[镜像加速器](https://yeasy.gitbooks.io/docker_practice/install/mirror.html)  
[How to Configure Network Static IP Address in Ubuntu 18.04](https://www.tecmint.com/configure-network-static-ip-address-in-ubuntu/)  
[netplan官网的配置示例](https://netplan.io/examples)

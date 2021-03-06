---
title: 试图抢救吃灰的树莓派
tags:
  - 树莓派3B+
abbrlink: '7042'
date: 2019-12-07 19:52:23
mathjax: true
categories:
  - 荒废的老本行
hide: true
---

$$
\frac{1}{3}
$$

自从来到沙河102网吧，都是在工作站/服务器/台式机上折腾，树莓派一直仍在清水河宿舍吃灰。今天回了趟清水河，把树莓派拿回来，试试能不能用，打算做备用测试机了。
<!-- more -->

## 装系统

从官网下载[^1], 我下的Raspbian Buster（SD卡就只有16G，不打算搞那些recommend software了），解压之后已经是3.56G了

然后用Rufus烧进SD卡（很多教程全是推荐Win32Diskimager，那个并没有Rufus好用），只选第一行的目标设备和第二行引导镜像，其他默认（卡里之前有东西的话还是用SD Formatter先刷一遍）

2020.3 更新

貌似树莓派官方出了一个写入SD卡的[工具](https://www.raspberrypi.org/blog/raspberry-pi-imager-imaging-utility/)，可以去试试


然后将SD卡查到卡槽里，再把显示器接上（我工位上的ROC显示器有HDMI接口和电源, 终于不用无显示器安装拿个笔记本上来就SSH折腾了）

不行，搜了一下看到config.txt文件里有

```
# uncomment to force a specific HDMI mode (this will force VGA)
#hdmi_group=1
#hdmi_mode=1
```

要把模式改一下啊

```
hdmi_group=2
hdmi_mode=58
```

这样感觉还是不方便，还要占掉我一个大屏。改回了SSH连接的方式

在boot目录新建SSH文件夹，然后新建一个wpa_supplicant.conf的文件配置WIFI

```json
country=CN
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
    ssid="你的WIFI名"
    psk="你的密码"
    key_mgmt=WPA-PSK
    priority=1
}
```

然后路由器找到ip，SSH连就行了，默认账号pi password：raspberry

发现这个树莓派真的鸡肋啊，扔着吃灰又可惜，不折腾硬件和IO不在乎功耗又不如有1080的工作站和20核CPU的服务器好（而且这俩还有公网IP和端口）

算了，每次折腾完了就后悔。
本逃兵不打算折腾硬件了，从医疗测控电路到stm32到树莓派的IO，都不太想碰了。时代变了，七八十年代的硅谷才是搞硬件的地方，二十一世纪是...反正不是硬件狗的世纪。251牢厂、加班狂魔联影、毁约的迈瑞，只想让人看了跑路。。。

服务器，启动。树莓派，关机。

## 注

[^1]: http://downloads.raspberrypi.org/
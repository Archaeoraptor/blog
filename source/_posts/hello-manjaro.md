---
title: 你好，Manjaro。再见，Manjaro。
tags:
  - manjaro
abbrlink: ff0e
date: 2020-01-17 10:41:09
---
本来我是想装个双系统的，但是想起教研室还有两个老主机，那就.....
<!-- more -->

~~装Arch是不可能装Arch的~~

## 更新———再见Manjaro

前一阵Manjaro社区出的争吵让我跑路了，Manjaro之前用的还比较舒服（当然直到现在都比较舒服），以后就不知道怎么样了。外面Arch社区骂声一片，里面还总在内斗，不断有人出走，前几周连jonathon都气走了（跑去隔壁endeavour os了）

jonathon的声明在[这篇帖子](https://forum.manjaro.org/t/change-of-treasurer-for-manjaro-community-funds/154888)

过程大概是这样的，philm（Manjaro的CEO，反正Manjaro社区的不少人都觉得此人非常讨厌）挪用社区赞助捐款2000欧元，用于给自己买的电脑。这次购买并没有跟掌钱管家jonathon商量，也没有在社区内征得同意。jonathon觉得按照规定这钱只能用于Manjaro项目的网站建设、项目推广之类的，不是给你们恰烂钱的，然后在论坛发帖子怼philm，然后帖子被删了一次，jonathon的管理员也被撤了。帖子下面众人纷纷表示philm不回应道歉以后再也不捐助Manjaro了，不少人表示以后再也不用Manjaro了，并考虑跑路到KDE Neon、Kubuntu、Arch、Endeavour OS。

philm的回应闪烁其辞，一直说事情不是大家想的那个样子的，但是大家怼他让他公布他和jonathon的完整聊天记录他又不肯。最后philm屁事没有，jonathon跑路去了Endeavour OS。

我之前从Ubuntu换到Manjaro，现在跟着jonathon跑到endeavour os（Antergos 的后继项目），因为懒得搞网卡驱动和g双显卡驱动了（这个Arch系的发行版显卡驱动、TLP和s-tui之类的也会附带装上，默认xfce桌面有很多系统自带的定制，我装的KDE，基本上相对于Arch没怎么魔改，附带了驱动个和很多常用的包）目前用着感觉还好，观察一下再评价。

## 安装

选Manjaro是因为对华硕老主板的硬件和驱动支持比较好，安装也比较方便（我可不想自己折腾Nvida驱动）。

（喜欢KDE桌面，又不想用openSUSE，Kubuntu之前试过体验不是特别好，转了一圈就Manjaro了）

去官网下了最新版的 Manjaro18.1.5 KDE桌面版本，还是拿Rufus做好启动盘用UEFI装，这里把空闲的老机子直接擦除原来已毕业师兄的win10（erase disk），选 no Hibernate（我不太喜欢systemd，就没设休眠）

看到选项里有个选择LibreOffice还是Free Office的选项，这里我选了FreeOffice试用，体验过两个月再回来填坑写体会。
（更新，还是WPS吧。。。）

可以断网安装。没有遇到问题，直接装好了，超乎想象的顺利。唯一的小插曲是装完系统后U盘只剩4M空间，用win10默认的格式化找不回来了，看了[这里](https://zhuanlan.zhihu.com/p/37772825)
网上教程已经很多了，我也不打算再写一篇了，下面仅作个人记录(其实并不怎么需要教程，过时得太快，还不如直接看官方wiki或者iso里打包的安装手册，不想看直接照着提示直接装都没问题，比Ubuntu好装太多了)
![manjaro-neofetch](hello-manjaro/neofetch-manjaro.webp)
内置python是3.8.1，我一时竟有些不习惯😂

## 配置

### 换镜像源

当然还是换成你电的，在`/​etc/​pacman.d/​mirrorlist`加上你电

### 输入法

更新，现在有一个官方打包的AUR： manjaro-asian-input-support，在community里面，有manjaro-asian-input-support-fcitx、manjaro-asian-input-support-ibus、manjaro-asian-input-support-ibus、manjaro-asian-input-support-fcitx5

```bash
sudo pacman -S manjaro-asian-input-support-fcitx
```

然后重启直接就能用了

KDE推荐fcitx，Gnome下大概ibus体验会好一点？（好吧其实我之前在Ubuntu Gnome上用的也是fcitx）

**下面这些都不用看了**

我用的~~sunpinyin~~libpinyin（一个比sunpinyin更新的输入法），googlepinyin也可以，sougoupinyin不推荐，对于它的流氓行为我实在无法放心（如果非要装，记得装fcitx qt4 或 qt5）

如果想要联网提示的功能可以装个cloudpinyin(要用archlinuxcn的源)

```bash
sudo pacman -S fcitx-im fcitx-googlepinyin fcitx-libpinyin fcitx-cloudpinyin
```

有个小问题，那个很多教程都写的/.xproflie, 官方wiki里新版是
~/.pam_environment里面加上那三行, fcitx官网上说是那个~./xprofile, 这里我放到.pam_enviroment里面了。

```log
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
```

然后点开右下角键盘的configure（配置）看不到输入法了，界面是空的，以为是系统语言非中文的问题，但是换了系统语言重启了以后还是不行。
运行了一下fcitx-diagnose, 报这个错误

```log
fcitx: BadWindow (invalid Window parameter)
```

感觉是看不知道哪个教程的时候在~/.xinitrc里面乱配置的时候出问题了

```bash
export LC_ALL=zh_CN.UTF-8
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
```

搜了一下是我弱智了，看[这里](https://bbs.archlinuxcn.org/viewtopic.php?id=1862)，照着教程乱搞的时候都没注意到。
这里把exec放到后面就行了，或者放到.xprofile里面

配置看这里
https://fcitx-im.org/wiki/Configure_(Other)/zh-hans

我看很多教程都是用的fcitx-configtool，这里推荐KDE桌面用kcm-fcitx进行设置，也可以直接编辑config文件

```bash
sudo pacman -S kcm-fcitx
```

### 软件包管理

#### pacman

装包首选pacman，然后是AUR，然后是其他pip、npm、deb之类的包（貌似Manjaro发布的20版pamac软件商店还支持了snap和flatpac......），然后是自己编译（对自己打包技术自信的编译狂魔请无视), 这样可以避免不少包冲突。
yaourt已经停止维护了[^1]，2020年了很多教程都还是这个，请自行换成yay[^2]等

另外多说一句，Manjaro很多地方和Arch还是不一样的，随便添加arch源或者archlinuxcn源是要出问题的

要加包优先从AUR里面加，尽量不要用archlinuxcn里的东西，Manjaro很多魔改和更新依赖版本速度和Arch不一样，非要使用archlinuxcn灵车漂移就把Manjaro设成unstable（并且出了事不要去archlinuxcn那里问，切记）

```bash
sudo pacman-mirrors --api --set-branch unstable
```

虽然名字叫unstable，但是它和Arch的stable基本上是类似的（它还要比Arch的stable慢三天），这意味着换成unstable再上archlinuxcn源灵车漂移翻车的几率会小一点。

Manjaro的包会比Arch的更新频率慢一点，Manjaro有三种选项，stable，testing和unstable，unstable比Arch的stable慢三天左右，testing比unstable慢，stable比testing慢。一般只是延后了几周而已，可以放心上unstable

#### yay安装AUR软件

这里用vscode举个栗子

不像apt有官方包，VSCode在文档上说可以用这个[非官方包](https://code.visualstudio.com/docs/setup/linux#_aur-package-for-arch-linux)

按照正常的AUR安装方式是这样的

```bash
sudo pacman -S git
cd /Downloads
git clone https://AUR.archlinux.org/visual-studio-code-bin.git
cd visual-studio-code-bin/
sudo pacman -U visual-studio-code-bin-*.pkg.tar.xz
cd ../ && sudo rm -rfv visual-studio-code-bin/
code .
```

这里我们有`yay -S visual-studio-code-bin`可以代替上面那一串命令
（pacman现在也有了包，直接pacman就好了`pacman -S code`）

#### pamac

另一种方法是直接从软件商店里点击安装，pamac或者octopi，其实也不算win10 巨硬store或者snap那种软件商店，大概就是一个pacman的GUI客户端吧（很多时候报错和提示并不好用，建议下载和更新还是用命令）

尤其不要同时开着pamac图形界面并打开pacman，pacman进程加锁出错系统容易崩了

### 网上冲浪

```bash
sudo pacman -S v2ray
```

把config.json放到/etc/v2ray下面
然后配置开机自启和启动（这里用systemd）

```bash
systemctl enable v2ray  #开机启动
systemctl start v2ray
```

然后配合switchyomega或者[mellow](https://github.com/mellow-io/mellow)或者[proxychains-ng](https://github.com/rofl0r/proxychains-ng)

如果还是习惯客户端可以试试v2rayA[^3]

### 其他

暂时没什么要说了，过一个月再来说体验吧，美化什么的也不想搞了，老主机带不动，我也老了，哎呀，对花里胡哨的欣赏不来了，一律默认配置

想起什么再说吧，ArchWiki真棒，看Wiki吧

#### 一些设置的小问题

装完输入法之后输入法调不出来了，发现Yakuake也调不出来了，感觉快捷键设置冲突，改快捷键设置就好了。

## 后记

总能看到一大批Arch玩家在Linux群里发洗手图疯狂劝退Manjaro用户
但是看到下面这篇，我决定还是用Manjaro吧。各种洗手劝退根本没给出让我换系统的强力理由
[Manjaro Controversies](https://rentry.co/manjaro-controversies) 中文版[[译] Manjaro 悖论](https://szclsya.me/zh-cn/posts/linux/manjaro_controversies/)
我还用windows呢，我还用Ubuntu呢，本弱智儿童欢乐多，你们随便怎么鄙视。自从当了硬件逃兵不折腾板子不玩单片机树莓派我连C都不怎么用，欢乐的拿python乱写脚本呢，节省心智的东西有时候真是好东西

人生苦短，
In the long run, we are all dead（虽然我并不喜欢凯恩斯主义）

我一直觉得商业化是个好事，尤其是操作系统这种东西。私以为大多数情况下知识产权是个好东西，收费也是个好东西，垄断是个坏东西。开源的一个重要贡献就是打破垄断。（虽然现在很多专利什么的都被玩坏了，特别是国内）
操作系统这种工业级的东西，商业化是正道，**微软的问题在于它的垄断，而不是商业化**

开源公司主动商业化上去抢市场抢用户，是好事哇
<!-- 
开源挺好的，但我不是个自由软件的教徒，也不是一个马教教徒，我对乌托邦的幻梦没什么兴趣

很多开源社区都有浓厚的抵触商业化的感情，但是现在经济并不好了，不少泡沫也要破了。软件传播成本接近零、容易破解、大家不愿意为软件买单，再加上近十几年国内外软件行业整体上升期，过的还不错，开源这么搞还能活得下去，靠着捐款和用爱发电也能过过日子。
看看硬件行业，开源早就没多少地盘了，一个比一个封闭。
你看论坛什么的，好多靠着版主和众筹能一直不盈利几十年活的好好的，烧钱的视频网站就不行，a站没几年就凉了。拒绝任何商业化却又需要很多投入又不直接带来太多价值的开源项目，只能越来越凉。
有比例巨大的开源用户都是被免费吸引而来的白嫖党，开源社区没有像邪教分权那样的筹钱能力，消耗很大的大项目，吃枣要凉。
个人英雄主义、理想主义、全球化的时代正在缓缓退去，孤立主义和贸易保护主义要来了，时间不等人。再这样下去，开源社区和小公司肯定要倒一大批，垄断集团只会越发强盛。
再多出多几个红帽那种的公司都算是烧高香了，市场总有人要抢，SUN那种好公司倒了，Oracle那种讼棍公司能把你们恶心死。很多人不喜欢Java就是让Oracle给恶心的，开源警察和自由软件警察再这么搞下去，好社区和好公司只会倒闭得更快。

扯远了，如果不怕滚炸，非重要工作环境，Manjaro是一个还不错的个人桌面版，大概像 Linux mint 那样值得一试。
 -->
（这Manjaro一股 Windows Vista 味，好看好用，bug多也能接受

## 时隔5个月的更新

终于返校了，这疫情闹得，哎呀

据说长时间不追滚动更新可能滚炸，不管了，硬着头皮试一下

直接

```shell
sudo pacman -Syyu
```

更新后重启显示密码不对，不知道是什么问题，试着重置了一下密码

（并不是我忘记密码了，这里据说前一阵不少人的KDE桌面密码出问题登录认证失败了）

重启按esc进grub

按e，在有`linux /boot /vmlinuz-linux`的那一行添加`init=/bin/bash`

ctrl+x出去输入`mount -n -o remount,rw /`给根目录读写权限，然后输`passwd`改密码，然后`reboot`重启

看起来没什么问题了，隔了几个月也没滚炸（至少看起来没有）

## 笔记本也换成Manjaro了

### 安装

小破本是灵越7591，不太想用win10了，Dell家的大部分系列都完美兼容Ubuntu（国外出售的7590的还有个developer版的装的就是Ubuntu，比预装win10少了一点微软授权费，可惜国内没有）

看到他们好多用黑苹果，我的本子不是4k，就没上黑果。（黑果Nvida独显也用不了）

Manjaro看到别人装上去还不错，索性就装Manjaro了（除了麦克风和显卡的驱动有点小麻烦，其他的蓝牙wifi触控板键盘休眠一切正常）Archwiki上也有同系列7590的参考[^5]

BIOS里关secureboot，改raid on为AHIC（不装双系统可以忽略，双系统记得改AHIC不然win10会蓝屏卡在启动界面）

装完发现Manjaro 20居然添加了snap商店，这是要跟Canonical同流合污？（逐渐Ubuntu化

KDE桌面内存占用在700M左右，还行

Ubuntu的显卡和外接显示器对Dell的本子都配适的极好，有Dell官方担保，不想折腾可以直接上Ubuntu（这款海外版有的就是预装Ubuntu，双显卡解决方案应该是Prime）

### 显卡驱动问题

不需要Nvida的卡就直接选free，装intel的核显就好了

想用Nvida的卡（好歹是一块1650ti不用太浪费了），装的时候Manjaro驱动选项可以直接选nofree，会自带

切换双显卡的配置可能导致看视频时卡成PPT，禁用硬件加速就好了，7591的丐版CPU硬解码也是带的动视频的（1080p没有任何问题）

### 麦克风声卡配置

之前7591的部分高通声卡驱动不行，麦克风用不了，后来有了一个开源的驱动[Sound open firmware](https://github.com/thesofproject/sof)，总归是能用了。

在linux kernel在5.6或以上，安装麦克风声卡驱动用的sof-firmware，可以解决声音和麦克风问题

```bash
sudo pacman -S sof-firmware
```

然后重启，我第二次重启后失效，喇叭和耳机都没有声音。按照wiki里面修改`/etc/modprobe.d/audio-fix.conf`(新建一个)，然后写一行

```config
options snd-intel-dspcfg dsp_driver=1
```

就行了。

或者在Grub的配置文件`/etc/default/grub`里设置

```config
snd-intel-dspcfg.dsp_driver=1
```

实在不行再试试内核参数(Kernel parameters)加上`snd_hda_intel.dmic_detect=0`

当然，仅仅是能用。声音效果不是太好，虽然灵越inspiron 7591/7590/7501 系列的扬声器声音本来就不太好，但是放点陶笛和埙的曲子还是有种凄婉呜咽感的，linux下用了sof之后感觉像是喇叭声。以后就只适合拿这个听刀郎了。

还有一个问题就是有的时候休眠挂起后再打开就没声音了，需要重启才行。暂时没找到什么好的办法，这个毛病之前win10就有

### 3.5mm耳机没声音

耳机孔没声音，wiki上的解决方案是

>In `pavucontrol`, try changing the Pulseaudio output profile from "Analog Stereo Output" to "Analog Stereo Duplex".

系统语言调成中文的用户点开系统设置-》音频音量设置=》音频=》高级=》内置音频，选模拟立体双工。

是的，这样就能用了（请忽略这个全损音质，耳机传来声音的时候我甚至觉得我的索尼耳机是拼多多10块钱买的）

### 耗电问题

这个目前我倒是没什么感觉，我的7591是97w的大电池版本，最好的省电办法是不用GNOME，KDE的掉电速度比GNOME大概能慢一半，轻度使用六七个小时没有太大问题。（跟win10上10个小时比还是有差距）。

极致省电可以上i3wm和禁用独显，不过7591电池够大，不像游戏本一样分分钟没电，倒是没有必要。

还有就是降压，给CPU降个50-75mV，效果感人。（对性能影响也很小）

然后安装thermald和tlp并设置开机自启动（tlp有个图形界面可惜是gtk的并且还在beta版，不太KDE用户建议用）

```bash
sudo pacman -S thermald tlp
systemctl enable thermald tlp
```

然后自行设置电池模式。

### 散热

当然7591/7590更要命的显然是捉急的散热，重度使用建议降压、换散热贴硅脂散热条、垫橡皮垫都安排上。有个s-tui可以查看CPU温度。

## 其他个人设置

### 常用软件

```bash
# docker
sudo pacman -S docker docker-compose
# 设置开机自启
sudo systemctl enable docker
```

#### 中英词典翻译和医学词典

这里我用的开源的GoldenDict配合离线和在线词典

```bash
sudo pacman -S goldendict
```

下载医学词典和英文词典，体验还是不错的，想要翻译功能可以添加有道和谷歌翻译

### 博客搬迁

先装node.js（这里我装的12的lts版）

```bash
sudo pacman -Sy nodejs-lts-erbium
```

然后从github拉取自己的博客备份repo

```bash
git clone https://github.com/archaeoraptor/blog.git blog
cd blog
```

另外我用的hexo-renderer-pandoc插件，这个插件要先装pandoc（不然会报错）

```bash
sudo pacman -S pandoc
npm install
git clone https://github.com/next-theme/hexo-theme-next.git themes/next
hexo g && hexo s
```

### windows软件解决方案

装wine体验还是不太好，虚拟机装个win7上跑qq和word，反正7591能加内存，加到32G就完事了，还不够就加64G（是的，这个系列支持64G内存，虽然说明书上写的只支持到32G）

### 网上冲浪

qv2ray还不错，electron-ssr装上v2ray插件也不错，但是electron内存占用太高了。喜欢web版的还有一个v2raya可以用。

```bash
yay -S qv2ray
```

装wireguard

```bash
sudo pacman -S wireguard-tools wireguard-dkms
```

如果用机场的话，可以装个clash，然后配置开机自启和规则（那个win版clash的图形界面真的反直觉反人类，相当难受，clashy也没好到哪去），貌似也支持套个tun/tap用来全局代理(似乎并不怎么好用)

```bash
sudo pacman -S clash
```

可以配置systemd让clash开机自启，[wiki](https://github.com/Dreamacro/clash/wiki/clash-as-a-daemon)上还提供了docker-compose和pm2的方法。(貌似pm2方便一点？尤其是装了nodejs的话，不过clash是go开发的官方推荐docker，不过docker和pm2要开机自启还是得用systemd)

上个月发布的clash1.0版本是和老版本不兼容的[^4]，AUR里打包的clash是不兼容的，所以要手动修改yml文件或者自己编译老版本的clash


>`Proxy`, `Proxy Group` and `Rule` is no longer used. They are now respectively `proxies`, `proxy-groups` and `rules

(修改的话一般只要修改`Proxy`, `Proxy Group` and `Rule` 这三个参数为`proxies`, `proxy-groups` and `rules就行了)

  设置完成后重启网络服务

```bash
systemctl restart systemd-networkd
```

### 中英文和字体设置

#### 把桌面文件夹改回英文

这里我用的[xdg-user-dirs-gtk](https://github.com/GNOME/xdg-user-dirs-gtk)，GNOME出品，KDE上也能用

```bash
sudo pacman -Sy xdg-user-dirs-gtk
export LANG=en_US
xdg-user-dirs-gtk-update
```

然后想把LANG变量调回中文的可以再调回来

```bash
export LANG=zh_CN.UTF-8
```

如果你用Dolphin文件夹管理器，你要把那几个桌面、文档、下载的路径在编辑里面手动改成英文

#### tty中文不显示

装点字体就好了，或者直接把全局的LANG=en.US

#### 复门等字体显示日文

最简单的做法是把系统装的语言包卸载了，只装中文字体包,见[Localization](https://wiki.archlinux.org/index.php/Localization_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)/Simplified_Chinese_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))。

或者在`\etc\locale.conf`加上`LANG=zh_CN.UTF-8`

然后刷新字体缓存。

```bash
fc-cache -fv
```



## 注

[^1]: https://wiki.archlinux.org/index.php/AUR_helpers
[^2]: https://github.com/Jguer/yay
[^3]: https://aur.archlinux.org/packages/v2raya
[^4]: https://github.com/Dreamacro/clash/wiki/Breaking-Changes-in-1.0.0
[^5]: https://wiki.archlinux.org/index.php/Dell_Inspiron_15_(7590)
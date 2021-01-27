---
layout: posts
title: AUR打包指北
tags:
  - AUR
  - Archlinux
categories:
  - 不务正业系列
abbrlink: create-aur-packages
hide: false
date: 2020-12-20 16:36:47
---
（长期施工中，未完待续）
<!-- more -->

{% cq %}
快来当AUR打包工具人/打包苦力吧(笑
{% endcq %}

## 大致流程

### 新建一个测试环境

在本机上直接打包可能会把你自己的机子环境搞乱搞崩，或者忽略了已经在自己电脑上装了了但是没有的依赖，又或者你想...
反正弄一个干净的测试环境是有必要的。

当然打包一些很简单、依赖很明确的东西，确定没必要也可以不要这一步。

### 编写PKGBUILD

最快的方法是去找找有没有和你要打包的东西类似的包，然后抄一下它的PKGBUILD。

### 检查和测试

先用namcap检查一下，有没有语法错误或者其他不符合规范的地方



### 编译

makepkg

### 上传和分享

#### AUR

如果想分享出来，你可以上传到AUR仓库里面。AUR帐号随便就能注册几乎没有审查的（所以传点恶意代码上去也能传的），但是尽量不要把有版权问题之类的东西传上去（虽然出了事应该是你自己负责）

新建一个账号，然后上传你的SSH公钥和密钥指纹。

#### 非官方软件仓库

或者你可以去其他非官方的仓库，这有一大堆：[Unofficial user repositories](https://wiki.archlinux.org/index.php/Unofficial_user_repositories_(简体中文))

中文用户人最多的是ArchlinuxCN，我猜你很熟悉这个了。

其他的还有chaotic-aur（里面的包不少，但是国内速度很慢，镜像都在国外，直接访问可能比从AUR拉下来编译还慢），arch4edu（一些教育软件和包，清华的弄得，国内速度还行），blackarch（这个包也很多，约等于arch系的kali源）

要添加到这些非官方仓库里，可以请求打包，或者跟仓库管理员py。

或者可以自建一个仓库。

## 一些工具

### namcap

用来检查打包是否正确

### devtools

可以干很多事情，最常用的是新建一个干净的环境用来测试你的包。

还有很多用法去看文档吧，

### debtap

大多数情况下可以快速把deb包转成aur包
有些deb的包或者rpm的包可以拆包然后重新手动打包一下。一般的用debtap可以解决大部分deb包，有些依赖可能没办法需要手动处理。

```bash
yay -Syu debtap
sudo debtap -u #同步数据库，可能比较慢
debtap -p XXX.deb # -p 选项生成PKGBUILD
## 然后根据提示输入包名和协议等
## 然后稍等一会，会比较慢
```

然后`pacman -U XXX.tar.xz`可以装到你的电脑上试一下。

### pip2pkgbuilds

将pip的包转成aur的包，这个工具很久没更新了（But it works）

## 自动化

人肉测试每个包然后传上去给大家用是最好的，不过有的上游更新太勤测试不过来，或者一些用
~~又或者有的时候你想让用户当小白鼠~~

## 一些小问题

打包的时候如果依赖AUR的包，在`makepkg -s`pacman是不会自动安装的需要手动安装

```bash
error: target not found: python-django-cors-headers
==> ERROR: 'pacman' failed to install missing dependencies.
==> ERROR: Could not resolve all dependencies.
```

如果依赖太多装不过来，可以加`makepkg -d`参数忽略依赖，然后对生成的`XXX.tar.xz`使用

## 练手推荐

上手AUR打包基本只需要一点bash脚本的知识就可以了。当然最好还要对打包的项目和所用的语言、框架、依赖比较熟悉。

（反正比deb和rpm的打包简单太多了，当时还在deb系的时候曾经在无聊的一天对deb打包好奇,在折腾了一天的打包之后，我放弃了
至于rpm打包，我当年看了这个[Linux 中 RPM 的构建与打包](https://developer.ibm.com/zh/articles/l-lo-rpm-build-package/)就放弃了,要打出一个好包看起来不会比deb打包简单）

官方的打包示例是要用到make编译c/c++包的例子，但是一些c/c++的包，运行环境和依赖会比较麻烦，而且在x86和arm等不同架构下面也会有很多问题。拿来熟悉打包流程对萌新不太友好。

练手打包建议用从自己最熟悉的项目和语言开始。如果你什么语言都不熟悉呢，推荐用一些主题或者可以用debtap之类的工具方便的转成AUR包的东西来熟悉一下打包。

有的系统主题、grub主题、输入法主题之类的包比较好打包，一般来说依赖也少，不会搞出什么大问题。而且主题的文件存放位置和PKGBUILD里面的其他东西可以找AUR里面打包好的其他主题参考，基本大同小异，上手打包不会有太太高难度和坑，不太熟悉流程的可以先用一些主题练手熟悉一下打包流程。（主要是这个依赖也少，破坏性比较小，萌新打出低质量的AUR包也不至于把装这个包的人一波带走闯下大祸）

当然打包不能包揽所有工作，一些设置需要用户手动启用或者修改的（比如一些配置，又比如systemd之类的daemon），比如我这里打包了一个GRUB的Cyberpunk主题，需要用户手动修改

```bash
vim /etc/default/grub
```

```bash
GRUB_THEME="/usr/share/grub/themes/Cyberpunk/theme.txt"
```

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

需要用户手动修改设置来启用（基本上主题、桌面挂件之类的包都是这么做的）

## 闲话

暑假的时候打了十几个野包，当时水平也菜，刚洗手变成Arch玩家没多久，打的十几个包都没放到AUR上丢人。虽然现在还是真蒟蒻在群里围观各路大佬卖弱不敢说话，看到中文的打包教程这么少，决定还是写点什么。

当时搜AUR打包相关的资料，发现相关的文章、博客、资料并不多（尤其是中文的）。中文资料保守估计不到Archlinux安装教程的百分之一，也不到deb、rpm、Flatpak打包教程的百分之一。

Archlinux安装弄的这么难初衷可能是为了筛掉一部分小白和伸手党，不过网上各路保姆级安装教程（一步一步手把手教的视频演示那种）和各种一键安装脚本和 Arch based 的发行版基本上快让[Arch的理念](https://wiki.archlinux.org/index.php/Arch_Linux_(简体中文))流产了(特别是广为诟病的Manjaro和网上一大堆不负责任的教程，一口一个适合小白，动不动就教人添加ArchlinuxCN源，搞得很多打包者不堪骚扰)

独立打一个包应该不会比独立装一个Archlinux难（不考虑Nvidia双显卡博通网卡阴间主板诡异驱动等....）。  
如果一个用户能参照ArchWiki独立装好、配置好自己的Arch，那就应该有能力自己打包。  
Arch没有Mac、Windows、ChromeOS那样的财大气粗的公司和掏钱买服务的客户，甚至不能跟RHEL和Ubuntu比，社区纯靠热情。而且由于deb系和rpm系用户众多，基本大部分软件如果支持Linux会给出deb或者rpm的包，有的可能会给个appimage的包，Arch的支持基本没太多上游会管，很多包都是Arch的维护者和用户自行编译打包的（不少还是deb拆包转的）。
伸手党太多而打包者太少，那就离凉凉不远了。  
下次看到没有包当伸手党可不好，没有包就自己打一个吧。  

## 参考和推荐阅读

[DeveloperWiki](https://wiki.archlinux.org/index.php/DeveloperWiki:Index) 里面关于打包的部分可以看看  
[Arch package guidelines](https://wiki.archlinux.org/index.php/Arch_package_guidelines) Arch官方的打包指南  
[PKGBUILD_(简体中文)]https://wiki.archlinux.org/index.php/PKGBUILD_(简体中文) PKGBUILD的简要介绍  
[Makepkg_(简体中文)](https://wiki.archlinux.org/index.php/Makepkg_(简体中文)) Makepkg也比较重要，有需要可以参考维基修改一下参数  
[给 Arch 打一个包 – Python 模块篇](https://felixc.at/2017/08/make-an-arch-package-for-python-module/) python的包经常没什么人愿意打（确实依赖比较麻烦，好在有肥猫，打了一大堆python包），希望多来点熟悉python的大佬  
[PKGBUILD参考手册](https://man.archlinux.org/man/PKGBUILD.5) Arch官网的参考手册，话说Arch最近新上的[手册索引](https://man.archlinux.org/)挺香的（btw I use tldr）  
[Building Packages on Arch Linux (Including the AUR)](https://www.vultr.com/docs/building-packages-on-arch-linux) Vultr的教程（我也不知道Vultr怎么会有这种东西）  
[Arch Linux 第一次打包就上手](https://junyussh.github.io/p/arch-linux-package-quick-start/)  新手可以看看这个
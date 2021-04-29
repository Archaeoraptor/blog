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

其实还是建议你去先看看官方文档

## 大致流程

### 新建一个测试环境

在本机上直接打包可能会把你自己的机子环境搞乱搞崩，或者忽略了已经在自己电脑上装了了但是没有的依赖，又或者你想...
反正弄一个干净的测试环境是有必要的。可以参考官方Wiki：[DeveloperWiki:Building in a clean chroot](https://wiki.archlinux.org/index.php/DeveloperWiki:Building_in_a_clean_chroot), 或者看肥猫这篇：[Arch Linux devtools 简介 – 在干净的环境里编译软件包](https://felixc.at/2017/08/introduction-to-arch-linux-devtools-build-packages-from-a-clean-chroot/)

当然打包一些很简单、依赖很明确的东西，确定没必要也可以不要这一步。

### 编写PKGBUILD

最快的方法是去找找有没有和你要打包的东西类似的包，然后抄一下它的PKGBUILD。

### 检查

先用namcap检查一下，有没有语法错误或者其他不符合规范的地方

```bash
namcap PKGBUILD
```

### 编译

```bash
makepkg -s
```

生成的`XXX.tar.xz`可以再用`namcap`检查一下

```bash
namcap XXX.tar.xz
```

### 上传和分享

#### AUR

如果想分享出来，你可以上传到AUR仓库里面。AUR帐号随便就能注册几乎没有审查的（所以传点恶意代码上去也能传的），但是尽量不要把有版权问题之类的东西传上去（虽然出了事应该是你自己负责）

新建一个账号，然后上传你的SSH公钥和密钥指纹。

用`git clone`拉取一个

```bash
git clone ssh://aur@aur.archlinux.org/your-package-name.git 
```

AUR软件仓库需要[SRCINFO](https://wiki.archlinux.org/index.php/.SRCINFO_(简体中文)),不然会拒绝你的提交

```bash
makepkg --printsrcinfo > .SRCINFO
```

#### 非官方软件仓库

或者你可以去其他非官方的仓库，这有一大堆：[Unofficial user repositories](https://wiki.archlinux.org/index.php/Unofficial_user_repositories_(简体中文))

中文用户人最多的是ArchlinuxCN，我猜你很熟悉这个了。

其他的还有chaotic-aur（里面的包不少，但是国内速度很慢，镜像都在国外，直接访问可能比从AUR拉下来编译还慢），arch4edu（一些教育软件和包，清华的弄得，国内速度还行），blackarch（这个包也很多，约等于arch系的kali源）

要添加到这些非官方仓库里，可以请求打包，或者跟仓库管理员py。

或者可以自建一个仓库。

可参考：

[利用 GitHub Actions 编译 AUR 包并建立自己的软件源](https://www.aloxaf.com/2020/06/build_aur_with_github_actions/)

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

将pip的包转成aur的包，这个工具很久没更新了（But it works）。这个工具有不少情况不会帮你自动处理，python的不少pip包本身也不太遵循pip的打包规范，要做不少手动检查和调整。

### npm-pkgbuild

[npm-pkgbuild](https://github.com/arlac77/npm-pkgbuild#readme) 一个将npm包转成aur的包，和上面那个类似，也是自动生成PKGBUILD。没用过不好评价。

## 自动化

人肉测试每个包然后传上去给大家用是最好的，不过有的上游更新太勤测试不过来，或者一些[用版本控制紧跟上游的包](https://wiki.archlinux.org/index.php/VCS_package_guidelines_(简体中文))
~~又或者有的时候你想让用户当小白鼠~~

可以试试Travis CI、Github Action 之类的CI/CD工具，不过最好还是更新之前人肉测试一下

### Github Action

Github Action的好处是可以白嫖（划掉

可以用Github Action自动生成PKGBUILD：
[publish-aur-package](https://github.com/marketplace/actions/publish-aur-package)

鸭鸭的[build-aur-action](https://github.com/DuckSoft/build-aur-action),用Github Action编译一个AUR包（有时候你不想在自己电脑上编译可以试试这个）

使用见：[GitHub Actions 打造 AUR 打包下载一条龙服务](https://viflythink.com/Use_GitHubActions_to_build_AUR/)

## 一些小问题

### AUR的依赖包makepkg无法自动识别

打包的时候如果依赖AUR的包，在`makepkg -s`pacman是不会自动安装的需要手动安装

```bash
error: target not found: python-django-cors-headers
==> ERROR: 'pacman' failed to install missing dependencies.
==> ERROR: Could not resolve all dependencies.
```

如果依赖太多装不过来，可以加`makepkg -d`参数忽略依赖，然后对生成的`XXX.tar.xz`使用yay或者pikaur之类的AUR helper

### git初次commit不包含SRCINFO导致拒绝提交

可以用`git filter branch`

（其实直接删了重新`git init`可能更快）

## 常见写法和命令

### install命令

用过`cmake`等编译工具的人应该对`make install`很熟了，它在PKGBUILD里面大概像这样

```bash
package() {
  make -C "$pkgname-$pkgver" DESTDIR="$pkgdir" install
}
```

在PKGBUILD里面，还经常用它把文件安装到制定目录，并指定权限（一般不用`cp`来干这种事）

```bash
package() {
  cd "$srcdir/$pkgname"
  install -vDm 644 ${pkgname%-git}{,.plugin}.zsh \
    -t "${pkgdir}/usr/share/zsh/plugins/${pkgname}/"
  install -Dm644 LICENSE "$pkgdir"/usr/share/licenses/$pkgname/LICENSE
}
```

将可执行文件和LICENSE分别放到指定目录，像LICENSE或者doc权限644就好了，其他的可执行文件酌情给个755权限。

如果有需要打印详细安装信息方便调试，可以加`-v`选项，像这样`install -Dvm755`

### 常见写法

获取Github Release

## 练手推荐

上手AUR打包基本只需要一点bash脚本的知识就可以了。当然最好还要对打包的项目和所用的语言、框架、依赖比较熟悉。

反正我感觉比deb和rpm的打包简单太多了，而且由于包管理的机制，虽然容易挂，但是依赖一般不会特别麻烦（除了有些要拆包的东西）

你看看依赖问题[都把隔壁deb系的老哥逼成什么样了](https://www.zhihu.com/question/291606128/answer/1194596591)。

可以看这个[Linux 中 RPM 的构建与打包](https://developer.ibm.com/zh/articles/l-lo-rpm-build-package/)感受一下打包流程
（当然rpm包用[fpm](https://github.com/jordansissel/fpm)之类的打包工具也没有那么麻烦）

官方的打包示例是要用到make编译c/c++包的例子，但是一些c/c++的包，运行环境和依赖会比较麻烦，而且在x86和arm等不同架构下面也会有很多问题。拿来熟悉打包流程对萌新不太友好。

练手打包建议用从自己最熟悉的项目和语言开始。如果你什么语言都不熟悉呢，推荐用一些主题或者可以用debtap之类的工具方便的转成AUR包的东西来熟悉一下打包。

有的系统主题、grub主题、输入法主题之类的包比较好打包，一般来说依赖也少，不会搞出什么大问题。而且主题的文件存放位置和PKGBUILD里面的其他东西可以找AUR里面打包好的其他主题参考，基本大同小异，上手打包不会有太太高难度和坑，不太熟悉流程的可以先用一些主题练手熟悉一下打包流程。（主要是这个依赖也少，破坏性比较小，萌新打出低质量的AUR包也不至于把装这个包的人一波带走闯下大祸）

而且他们不需要编译，也没有依赖报错，总之非常的适合熟悉一下PKGBUILD的流程

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

小白倒没什么，主要是被伸手党搞怕了。

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

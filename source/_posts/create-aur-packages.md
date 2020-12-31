---
layout: posts
title: AUR打包指北
date: 2020-12-20 16:36:47
tags:
 - AUR
 - Archlinux
categories:
 - 不务正业系列
abbrlink: 'create-aur-packages'
---
为什么arch包多呢，~~因为打包方便~~，因为肥猫
<!-- more -->

{% cq %}
快来当AUR打包工具人和ArchlinuxCN打包苦力吧(笑
{% endcq %}

暑假的时候打了十几个AUR野包，当时水平也菜，刚洗手变成Arch玩家没多久，打的十几个包都没放到AUR上丢人

上手AUR打包基本只需要一点bash脚本的知识就可以了。

（反正比deb和rpm的打包简单太多了，当时还在deb系的时候曾经在无聊的一天对deb打包好奇,在折腾了一天的打包之后，我放弃了
至于rpm打包，我当年看了这个[Linux 中 RPM 的构建与打包](https://developer.ibm.com/zh/articles/l-lo-rpm-build-package/)就放弃了,要打出一个好包看起来不会比deb打包简单）

当然最好还要对打包的项目和所用的语言、框架、依赖比较熟悉。

有的系统主题、grub主题、输入法主题之类的包比较好打包，一般来说依赖也少，不会搞出什么大问题。而且主题的文件存放位置和PKGBUILD里面的其他东西可以找AUR里面打包好的其他主题参考，基本大同小异，上手打包不会有太太高难度和坑，不太熟悉流程的可以先用一些主题练手熟悉一下打包流程。

比如我这里打包了一个GRUB的Cyberpunk主题

```bash
vim /etc/default/grub
```

```bash
GRUB_THEME="/usr/share/grub/themes/Cyberpunk/theme.txt"
```

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

一些c/c++的包，运行环境和依赖会比较麻烦，而且在x86和arm等不同架构下面也会有很多问题。Arch的依赖
有些deb的包或者rpm的包可以拆包然后重新手动打包一下。一般的用debtap可以解决大部分deb包。


## 参考和推荐阅读

https://junyussh.github.io/p/arch-linux-package-quick-start/
https://wiki.archlinux.org/index.php/Arch_package_guidelines
https://wiki.archlinux.org/index.php/PKGBUILD_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#license
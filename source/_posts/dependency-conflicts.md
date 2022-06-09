---
title: immutable思路面对依赖冲突：Nix包管理器
tags:
  - dependency hell
  - functional
  - Nix
abbrlink: dependency-conflicts
categories:
  - 杂七杂八
date: 2022-05-31 21:56:02
---
一直没有什么处理依赖问题的好思路，后知后觉发现Nix包管理器。要是当年我还沉迷lisp的时候真的注意到这个，说不定现在人已经跑到NixOS去了
<!-- more -->

大多数发行版的版本是几年或者几个月一个大版本，然后维护对应版本的依赖和软件（一般依赖不怎么大更新，打补丁）， 比如Ubuntu、红帽这些商业发行版，没什么大问题基本不怎么动。不过维护这么多版本需要耗费不少的精力，而且由于rpm和deb打包比较麻烦，包不太多。

几年前我用deb包管理器的时候遇到依赖问题几乎是家常便饭，然后动不动就用第三方的方式装，比如make编译、比如野路子`curl XXX install.sh && ./install`， 于是环境变得乱糟糟的。后来Ubuntu开始推snap。于是我跑路到了pacman，后来就一直用Arch了，还自己打了点AUR的包。

pacman包管理器没有deb系那么多功能，不过我觉得好就好在简单，这样打包门槛低，促进了AUR的繁荣。不管AUR的PKGBUILD再怎么质量差再怎么野，至少也比找不到deb或者rpm的包然后纷纷make或者`install.sh`好多了

另一种思路是尽量不要动态链接库，代价是打包出来的体积会比较巨大。依赖？没有依赖不就不冲突了吗？统统静态链接，不过问题是打出来的包比较大。另一个好处是可以顺便做一个虚拟的目录，把更多的东西都给每个软件单独做一份，做出类似沙盒隔离。比如flatpak，这样打出来的包比较容易跨发行版，也不太容易遇到依赖找不到或者依赖版本冲突的问题。

Arch已经足够让我满意了，包多，自己打包也简单。不过每次多版本共存的时候还是有点别扭，难道就真只能所有人尽量统一版本往前滚了吗？多版本怎么共存呢？我不想要pyenv、archlinux-java、gvm、nvm、conda这些东西，这些东西应该由一个全局的包管理器去处理。

理论上如果绝大部分上游应用和库的向后兼容做的足够好，那确实不太需要太关心多版本并存的问题，每个大版本打一个包就好了。比如go，我从1.14升到1.18，原来的代码和库基本能跑的依然能跑。但是python这样的，好家伙，python2到python3升级大量beraking changes就算了。python3.5、python3.6、pyhton3.9、python3.10这样的小版本号动不动就不兼容，然后一堆依赖出问题了。python那一堆包的用法和函数还天天改，动不动版本更新之后原来import的包不能跑了，你要改函数名。尤其是一些深度学习的包和django的一些东西、好端端的为什么要改来改去？还有grpc也是，不遵循语义化版本号，我好几次滚arch都是某个应用依赖grpc然后出问题了，版本又不兼容。

除了兼容问题还有另一个问题不大的小问题，就是不同包的文件冲突，比如bear和interception tools， 都在arch的community包里面， 都有一个`/usr/bin/intercept`。当然一般软件命名不一样没这个问题，或者把这两个包标为`conflict`。那有没有办法不改名也能用呢，那能不能给每个包一个自己的专属目录。这个问题和上面那个多版本并存的问题是很像的，同时安装两个版本也会在`/usr/bin/XXX`等各种重名目录下面相同文件冲突。

最近才看了一下Nix包管理器，之前一直觉得这个玩意就是茫茫多包管理器中的一个，和别的也没多少区别。一看才发现这不就是我要找的吗？

>Nix is a purely functional package manager. This means that it treats packages like values in purely functional programming languages such as Haskell — they are built by functions that don’t have side-effects, and they never change after they have been built. Nix stores packages in the Nix store, usually the directory /nix/store

Nix包管理器将包放在`/nix/store`下面，然后根据sha256 的 hash去找版本。这样就实现了多版本共存，然后这个在打包的时候每个包是显式声明依赖的，不会出现打包的时候漏写了哪个依赖然后跑到别的电脑上寄了的情况。就算系统上有需要的依赖也不自己去找，需要显式声明。这样就可以确定完备的依赖关系，然后对一些嵌套的依赖进行推导。

然后有了这个声明式（Declarative）的配置，我们可以还原以前的版本（有点像git）

Nix保留每一个包版本，像python3.9和3.10如果有依赖或者显式声明都会安装，然后依赖这些的包分别根据hash和打包声明去找相应的版本。这里是只要有一点不同就是一个新的版本，就算3.9.1和3.9.0也不一样。这样就不怕一些包不管语义化版本随便乱改API或者ABI了。

~~那这么多包，体积岂不是很大？是的~~

以Nix包管理器为发行版的NixOS，是需要你去写一个全局的配置文件`configuration.nix`，就像dotfiles一样。

不过NixOS应该也确实没什么太多人用，一直不怎么火。毕竟为了一个Nix包管理器再多学一门Nix语言，然后为了这盘醋包了这盘饺子，总感觉有点代价太大了。当然也可以不学，有一个Guix是直接用lisp的。不过这个更加清蒸，一些非自由软件你要自力更生。

NixOS另一个巨大的问题是不兼容FHS，就是`/usr/bin`之类的那一套目录。然后很多在别的发行版跑的好好的应用由于目录问题就跑不起来了，需要打一大堆patch。而且Nix的包现在太少了，各种问题需要自己解决。虽然安装门槛并不比Arch高，但是打包门槛和工作量我觉得比Arch高不少。于是我在虚拟机里面玩了一圈又滚回了Arch，又不是不能用.jpg （其实再不删掉继续玩下去我硬盘不够了）

最近听说deepin要脱离deb做自己的包管理器了，祝他们好运吧。deepin这么做会不会加剧发行版包管理器碎片化已经无所谓了，比起只顾着套壳圈钱的产品，折腾出点什么总不算太坏。

## 链接

[The Purely Functional Software
Deployment Model](https://edolstra.github.io/pubs/phd-thesis.pdf)  
[金枪鱼之夜：Nix - 从构建系统到配置管理](https://tuna.moe/event/2021/nix/)  
[You Are Most Likely Misusing Docker
](https://www.mpscholten.de/docker/2016/01/27/you-are-most-likely-misusing-docker.html)  
[Nix, the purely functional package manager](https://github.com/NixOS/nix)  
[Nix Manual](https://nixos.org/manual/nix/stable/)
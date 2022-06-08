---
title: immutable思路面对依赖冲突：Welcome to the hell
date: 2022-05-31 21:56:02
tags:
- dependency hell
- functional
- Nix OS
abbrlink: 'dependency-conflicts'
---
一直没有什么处理依赖问题的好思路，后知后觉发现Nix包管理器。要是当年我还沉迷lisp的时候真的注意到这个，说不定现在人已经跑到NixOS去了
<!-- more -->

依赖？没有依赖不就不冲突了吗？统统静态链接，不过问题是打出来的包比较大

之前一直觉得这个玩意平平无奇，就是茫茫多包管理器中的一个。

不过NixOS应该也确实没什么太多人用，一直不怎么火。毕竟为了一个Nix包管理器再多学一门Nix 

## 链接

[The Purely Functional Software
Deployment Model](https://edolstra.github.io/pubs/phd-thesis.pdf)  
[金枪鱼之夜：Nix - 从构建系统到配置管理](https://tuna.moe/event/2021/nix/)  
[You Are Most Likely Misusing Docker
](https://www.mpscholten.de/docker/2016/01/27/you-are-most-likely-misusing-docker.html)  

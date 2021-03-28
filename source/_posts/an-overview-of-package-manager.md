---
layout: drafts
title: 一个好的包管理器应该是什么样子
date: 2021-03-20 15:00:25
tags:
- package
abbrlink: 'an-overview-of-package-manager'
---

在很久很久之前，人们把，欢乐滴make编译，后来东西多了，依赖地狱就来了
<!-- more -->

## 包管理器的功能

方便安装
解决依赖问题
方便版本升级
卸载后比较干净

像应用商店一样的东西

## 一些取舍

### 要不要去中心化

有一个大一統的网站的，比如npm,比如pypi，
去中心化的像Golang、deno

### 版本管理策略

一种是维护多个版本，用户自行选择使用哪个版本

另一种是直接拉满

### 打包粒度应该有多细

打包粒度粗一点，所花费的不过是

我倾向于都2021年了，固态硬盘的价格都不到1块钱1个G了（个人用的，企业级的硬盘会贵一些），真没必要省这点存储空间

### 

## 我与包管理器

### 不知有之

很早很早的时候上中学微机课写过basic，然而当时只是那种不超过几十行的玩具代码，根本不知道项目、工程、workspace为何物，更不要谈包管理器这种东西。当时只用过 Windos XP 和 Windos7， 以及诺基亚手机，连“软件商店”、“应用中心”都没有。

### 初见

当时大二还是大三开始用Ubuntu（在虚拟机里）和51、stm32、树莓派、Xilinx的板子。当时对包管理器的认识仅限于`apt install`和`dpkg -i`。那时候主要在写c和VHDL/Verilog, 并不怎么用包，基本都是c/c++用到什么库就直接下载下来放到目录，用到什么芯片驱动、PCB的封装库也都是手动下载的。（说起来封装库什么的Altium Designer和Vivado应该可以直接勾选自动下载的，可惜你电在清单上并没有正版，我用的都是断网的盗版）

然后发现`apt install`还挺好用的，至少当时感觉比提心吊胆的点击exe安装包然后弹出

后来大三的时候发现python真是好用啊，写起小脚本来比c舒服太多了，用Django糊了一个XX管理系统的课程大作业，每天在win10上跟`pip`作斗争。由于大作业的前端没用上Vue、React之类的脚手架，所以逃过了`npm`包的毒打。后来又乱入了机器学习的坑，开始整一些掉包调参的垃圾（理解一下，要恰饭水论文的嘛），在win10上被迫接触了anaconda这个东西（当时没直接主力换到Linux,还在做一些硬件相关的东西，好多EDA和仿真软件并不支持Linux）

不知道为什么我第一次接触anaconda就特别讨厌这个玩意，后来看到anaconda因为版权取消了包括tuna在内第三方镜像源的分发，更讨厌了....

### 开始怀疑

后来在Ubuntu、Debian、Raspbian上多次遇到依赖问题，搞得头都大了
然后后来又把wordpress的博客换成了hexo的，第一次见识了几个G的npm包

### 见得多了

后来出于好奇看了看moderm c++ 和 rust，可能c写习惯了，
维护教研室的服务器接触了`rpm`包和老CentOS的`yum`

### 成为一个很菜的打包者

当时曾经对deb包好奇，



## 参考文章和推荐阅读

[$GOPATH 耦合之殇](https://blog.wolfogre.com/posts/why-no-gopath/) 
[从 goinstall 到 module —— golang 包管理的前世今生](https://blog.wolfogre.com/posts/golang-package-history/) 这两篇是wolfogre写的，写得很用心，推荐
[包管理器的前世今生](https://forelax.space/lets-talk-about-package-manager/) 讲homebrew和Ruby的....很不巧我既没有Mac也没用过Ruby

https://zhuanlan.zhihu.com/p/83698275
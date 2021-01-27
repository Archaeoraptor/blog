---
title: Linux桌面的定制和魔改
tags:
 - KDE
 - 魔改
abbrlink: 'customize-your-linux-desktop'
---
怎么是个人的桌面都比我好看

<!-- more -->

## 前方预警

又是不务正业的一篇，这里不想讲那些仿Mac和仿win10，也不会讲latte-dock之类已经人均试过的东西。

跑过来寻找桌面美化朋友看到这里直接退了去找其他手把手的教程好了。（其实直接抄dotfiles更好）

由于我手头的Linux几乎全重装成了Arch，本文的内容会包含更多的Arch和KDE的东西，其他很多不同的发行版和桌面环境可能不适用

很多美化、定制、魔改方法会导致你的系统不稳定、黑屏、崩溃，请量力而行

## 关于DE

GNOME最好的发行版可能是Fedora，KDE最好的发行版是OpenSUSE

## 关于WM

极简爱好者，直角爱好者，需要省电

## 字体

Linux下的字体渲染在1080p的低分屏下面效果比win10好（特指中文）
4k屏等高分屏下面win10的字体渲染和苹果的苹方字体渲染效果比Linux下的字体好（也指中文）

很多字体不包含图标，你可以自己给它们打patch

你可以用gucharmap，

很多常见图标可以用nerd-fonts，它打了很多font-awesome和material-design的图标的patch


## 参考和推荐


https://github.com/ryanoasis/nerd-fonts/releases
http://suckless.org/ 精简
http://dotshare.it/ 一个分享dotshare的网站  
https://www.reddit.com/r/unixporn/wiki/info/postofthemonth UnixPorn的每月桌面，神仙打架

## 关于本人

### 个人喜好

我的笔记本和台式都用的KDE，所以很多魔改是在KDE上操作的。如果有空了，可能会给我吃灰了很久的闲置~~树莓派~~发霉派换成Archlinux ARM和i3wm桌面，到时候会更新一点WM(Window Manager)的内容。

此外，我喜欢直角~~对，就是本博客用的Next主题这样的直角~~  
用不习惯GNOME和Mac的大圆角，chrome现在的圆角也不习惯，还是喜欢Firefox和老版chrome的直角，新edge那种不那么圆的角也能接受。

很不喜欢 Material Design，更喜欢锋棱而非圆滑

更喜欢拟物而非扁平

喜欢像素风，然而找不到太多制作精良的像素风画作和主题

喜欢金属和机械风格，然而非常讨厌高达机甲

及其讨厌后现代的一些流派

### 太长不看

总之就是本人口味挺挑剔，然而自从幼儿园以后画画水平一直在退步，又没有太多时间

“将就用呗，又不是不能用”
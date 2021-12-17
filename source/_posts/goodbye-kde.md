---
title: 从KDE迁移到i3，顺便说说Linux桌面(DE)和窗口管理器（WM）
date: 2021-11-13 22:25:47
tags:
- KDE
- i3wm
- kiss
abbrlink: 'migrating-from-kde-to-i3'
---
逐渐开始认同Keep it Simple, Stupid
<!-- more -->

前一阵不是KDE 25周年吗，然后KDE推出了25周年纪念版，从这开始的几个版本，小问题不断。迫害GNOME是KDE群友的传统艺能，然而，最近似乎KDE用户受到的迫害比GNOME还多。KDE怎么会变成了这个样子？ 于是决定从KDE先迁移到i3之类的wm去。（当然，最主要的原因是我逐渐不需要一个完整DE的那么功能了）

## 从KDE说起

KDE一度是我最看好的Linux桌面，功能该有的都有，当时也比较稳定，那时我觉得最近3年应该不用换到其他的桌面去了。
KDE为什么是神，首先，要从Konqueror和KHTML的荣光说起...
KDE4时代虽然好看，但是因为不稳定而饱受诟病。自从KDE5以来，KDE也稳定了,
直到，KDE 25周年特供版......每次更新总有一些新问题，又开始不稳定了.....

### 防火防盗防抄袭的配置文件

想抄一下我的KDE的桌面设置？不好意思KDE的设置我自己也不知道它给我保存到哪里去了。

KDE的配置文件在`config`里面扔的到处都是，实乃居家旅行防偷配置利器。

`.config`目录下面有一个有一个小写的`kde`目录，还有一个大写的`KDE`目录,还有一个`kde.org`目录，下面有`systemsettings.conf`和`plasmashell.conf`这样的配置文件。如果你是祖传home目录，那八成还有个`kde4`。如果你是比较新的用户，你会开心的发现`~/.config`下面没有`kde4`，是的，他跑到`~/.kde4`这来了,`~/.kde4/share/config`下面有一些kde4的配置。
然后，`~/.config`下面，还有`plasmarc`，`kded5rc`，`plasmashellrc`，`kdeglobals`，天知道它们有什么区别。
然后是一堆散落在`.config`下面的KDE全家桶应用, 比如`kalarmrc`。这次看起来挺合理吧，你看别的应用好多不也都把配置文件扔在`~/.config`下面。

别急，比如KDE出品的和notepad3、vscode类似的文本编辑器kate，在`~/.config`下面有`katerc`, `kateschemarc`, `katevirc`等等，跟VSCode一个json文件比起来拆的还是太散了。在比如elisa这个音乐播放器，有一个`elisarc`，还有一个`kde.org/elisa.conf`。

更麻烦的是这些KDE的设置和其他应用都混在一起了。有一天你的配置崩溃了，KDE起不来，你想全部恢复默认设置，又不能直接全删了来恢复默认设置。（比较好的办法是挑出不是KDE的设置或者）。比如`trashrc`是KDE的配置文件，它看起来跟KDE毫无关系。

当然有一些第三方工具可以保存KDE的设置，比如`konsave`，我还给他打过一个AUR包。不过用了两天我立刻发现这并不能解决问题，我想抄别人的配置去改一改还是要打开KDE系统设置的图形界面。某一部分微小的改动，我还是要从茫茫的配置文件里面找到它们，KDE官方有没有说明，结果就是改起配置文件比手动在设置里面调整慢。

当然KDE有一个kcmshell5的命令工具可以在命令行配置，不过我想正常人没有用这个的。你以为他是命令行工具，实际上你执行他，他给你弹出一个和系统设置一样的图形界面。

估计KDE也没想让你使用`dotfiles`或者直接编辑配置文件，都给我用鼠标在设置里面点击选择的你设置！

### 小而美的Baloo

>baloo 是张小龙开发的吗

哎，这index索引文件怎么50多个G？哎，我桌面怎么突然卡死了？哎，我CPU怎么突然100%了？哎，我内存怎么占用飙升？

Baloo这个用过KDE的应该都有体会，体验大概就是Windows装了个流氓全家桶的感觉。 

一个索引服务, 磁盘写入远超读取是怎样一种体验？

我建议每个每个KDE用户装上KDE立刻关闭Baloo，我也建议有良心的KDE发行版在KDE的默认设置里面禁用Baloo

```
balooctl suspend
balooctl disable
balooctl purge #delete the index
```

如果你还想要一个搜索工具，推荐rg、fd、fzf。
如果你想要一个Baloo那样带索引的，试试plocate

### 混成器

混成器（Compositer）造成了KDE4时代KDE不如GNOME稳定的传说，当时OpenGL导致画质撕裂。据说当时大部分不稳定的锅都是混成器的，KDE其他的部分还是挺稳定的。
就在10月份，KDE将Compositer的选项

Vulkan也不知道要等到什么时候，据说要KDE6了，到时候再试试KDE吧。

## 使用平铺wm

最近一年来我对Desktop Enviroment的需求逐渐降低，很多操作都从鼠标点击过渡到了命令操作。我在KDE桌面上操作窗口也从鼠标点击变成了Win+上下左右平铺、Alt+Tab切换窗口、Win+Tab切换桌面。之前非常喜欢的yakuake也被我用tdrop+alacritty替代了。那些标题栏、桌面小组件、任务栏逐渐开始变得鸡肋，而且我也不想要那么多的桌面过渡特效。为了更好地移动窗口我试过krohnkite（一个模仿dwm操作的脚本）

在发现自己的需求基本上wm都能满足之后，好像没有什么用DE的必要了。

平铺的好处是比较充分的铺满整个桌面，而且可以省掉标题栏，对于比较小的笔记本屏幕可以省空间。（笔记本上用wm还比较省电，轻松撑过一整天）

关于wm的介绍可以看这个视频：https://www.youtube.com/watch?v=Api6dFMlxAA

dwm是类似栈的操作，而i3wm的窗口是一棵树。不过轻度使用应该区别不大，我几乎只用将屏幕两等分或者四等分、或者竖着三等分。

### 真的一定要那么依赖鼠标和GUI吗

早些时候人们用键盘的快捷键和命令和计算机打交道，鼠标和GUI界面反而是比较后来的事情了。据说微软当年为了让用户习惯鼠标操作，推出了扫雷、空当接龙和蜘蛛纸牌让大家玩

我从第一次用Windows XP开始就是拿鼠标在图形界面上点点点，在小时候的相当长一段时间内键盘打字都不熟练，后来微机课教的也是Office、Flash、photoshop等拿着鼠标在一堆设置按钮里面选来选取的那种，甚至外的游戏也都是pvz、愤怒的小鸟等鼠标操作居多

### i3的一些问题

不管是i3还是sway，或者dwm之类的，都对浮动窗口支持很差。其实挺难受的，比如沙拉查词之类的翻译软件或者浏览器小窗播放、又比如输入法那个小菜单栏（虽然现在fcitx5没有浮在外面的菜单栏），不太适合平铺。

i3的浮动窗口必须置于平铺的下面，挺难受的。

另外这些平铺桌面

### 安装

```bash
# 我装的cn源里面那个功能最多的开发版本，如果没什么特殊需求可以直接装i3wm或者i3-gaps（支持窗口缝隙）
# yay -S i3-gaps
yay -S i3-gaps-next-git 
yay -S picom 
# 用来设置外观、锁屏什么的
yay -S lxappearance 
```

还有一些fcitx5的configtool、qt界面的设置软件，我用KDE的时候都装过了，这里略过。

### polybar作为状态栏

polybar比i3bar功能多一点而且好看，但是占用也高

polybar自己一点一点配置会很麻烦，比较懒的我直接用的这个：[polybar-themes](https://github.com/adi1090x/polybar-themes)，一个awesome wm的polybar设置脚本，有相当多的主题可供选择。

```bash
git clone --depth=1 https://github.com/adi1090x/polybar-themes.git
cd polybar-themes
chmod +x setup.sh
./setup.sh
```

然后选一个你喜欢的主题改一改就好了。

### picom配置

其实我对阴影特效、模糊透明毛玻璃、圆角都没有什么需求。装picom主要是解决一下画面撕裂的问题（毕竟Nvidia f**k you, 混成器不要是不行的）

picom可选`xrender`和`glx`，以及``
如果要用自带的毛玻璃效果请使用cn源里的`picom-git`并开启experimental-backends。然后在i3的`config`设置里面也加上`picom --experimental-backends`

### i3lock-color锁屏

```bash
yay -S i3lock-color
```

### gtk主题配置

我用的是capitaine-cursors arc-gtk-theme。这个用lxappearance设置。

### HiDPI配置

大部分应用在`~/.Xresources`里面配置一下就可以了。但是polybar、rofi这几个都不吃

polybar

config里面

```
dpi = ${xrdb:Xft.dpi:-1}
```

rofi需要自己一点一点都调了，字体差不多16px，然后圆角和图标那都调大一点。绝大部分rofi主题都是1080p的，要自己调大小。

## KISS

>Keep it simple, stupid

现在越来越喜欢简单的东西了


>keep it sufficiently sophisticated


## 其他的选择

1.Wayland

I use Nvidia...., so Nvidia f**k

最近某些媒体宣称KDE+Wayland+Nvidia已经差不多了，然而前几天我试了一下KDE的wayland还是有不少小问题的。暂时没有什么迁移到wayland的动力。而且不少应用在xwayland下面的HiDPI体验不是很好。最重要的是alacritty这个东西，在wayland会卡。而我现在对

2.Sway

原因Wayland，我用Nvidia显卡。
Sway现在输入法问题基本可以了，剩下的是xwayland在HiDPI有一点不清晰和一部分x的应用我比较习惯所以暂时不打算迁移到wayland

https://github.com/swaywm/sway/pull/4740
https://github.com/fcitx/fcitx5/issues/39

3.Wayfire

Wayland的原因同上。 
Wayfire主要还是堆叠窗口的操作，特效和流畅程度看宣传和演示视频可能是Wayland里面最棒的一个了，响应丝滑、特效也很棒。
不过Wayfire的那些炫酷的桌面效果我不太想用，我自己大概有blur和活动窗口变亮一点就可以了。  
Wayfire关掉点特效用来做平铺和堆叠的混合桌面应该不错。因为像i3等对堆叠窗口的操作没有那么丰富，比如堆叠窗口只能放在平铺窗口的上面。

Wayfire目前是0.7版本，最近我看到依云在试水，HiDPI和输入法的问题patch得差不多了，我再等等吧。等1.0正式版，我迁移到Wayland和Wayfire。

4. KDE的平铺选择？

如果你对平铺操作的要求不太高，那么KDE自带的快捷键差不多就够了，如果要更多功能，推荐[bismuth](https://github.com/Bismuth-Forge/bismuth)

## 链接

[迫害GNOME的频道](https://t.me/ArchCNKDEVSGNOME) （仅供娱乐）
https://invent.kde.org/plasma/kwin/-/merge_requests/1088 KDE在这个Merge移除了XRender混成器
https://mail.kde.org/pipermail/kwin/2021-June/005232.html 讨论移除XRender的邮件列表
https://www.reddit.com/r/kde/comments/qamlfd/was_xrender_compositor_removed_in_plasma_523/ reddit上的移除XRender受害者（这位是i卡，不是N卡）

Vulkan
http://blog.davidedmundson.co.uk/blog/running-plasmashell-with-vulkan/
https://github.com/KDE/kwin/commit/811beb94e0a7dd5666906b07a51a84efe5f1bb53

Nvidia的问题（不是很懂为什么移除xrender）
https://www.reddit.com/r/kde/comments/qskvr1/screen_flicker_problem_with_144hz_on_kde_neon_any/

i3wm配置参考：

[I3wm 配置思路](https://zjuyk.gitlab.io/posts/i3wm-config/)

Polybar配置：

https://github.com/adi1090x/polybar-themes
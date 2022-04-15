---
title: Vim的寄存器与复制粘贴
date: 2022-04-01 20:19:37
abbrlink: 'vim-registers-copy-and-paste'
tags:
- vim
categories:
- 不务正业系列
---
清明节假期写点Vim相关的东西，说一下寄存器。在此之前我眼中的复制粘贴从未如此复杂，就像我从未
<!-- more -->

另一个比我想象中复杂和有用的是Vim的undotree，有点git那种branch的味道（装上gnudo插件就更像了）。

## Linux下的复制粘贴

首先说一下linux下面的复制粘贴，一般linux桌面的复制粘贴是`x11`负责的，复制粘贴的时候和`X Server`通信（一般管复制粘贴叫selections），有两个常见的selections：

PRIMARY：鼠标左键选中复制，鼠标中键粘贴
CLIPBOARD： `Ctrl + Shift + C`和`Ctrl + Shift + V`

这两个保存的内容是不同的，而且当你复制粘贴的时候两个应用必须同时开着，x11本身不全局保存复制的内容，应用关了就没了。所以如果你想要只有一个全局的剪切板，你得装一个`clipboard manager`（大多数桌面都贴心的自带了，可惜某些suck的wm并没有这个东西，曾经搞得我好几次弄丢了复制的东西不知道怎么回事）

推荐鼠标操作的时候尝试适应鼠标中键的操作，这样只要一只手控制鼠标就行了，比一只手鼠标一只手`Ctrl+C Ctrl+V`节省了一只手。在x11下的terminal中复制东西我一般用`xclip`。

## 如果你只是简简单单复制粘贴

如果你不得不用vim，那还是鼠标选中和的复制粘贴更适合你，进入insert模式粘贴即可。简单的命令可以在normal模式下使用y和p即可（yank和paste， c命令被change占了）。y和p两个命令和其他的命令组合，可以覆盖绝大多数的用途。  

## 精细组合操作

比如`yw`复制一个单词，`yy`复制一行，`y$`复制当前光标到结尾, `y^`复制当前光标到开头， 同理，`yG`复制当前行到结尾。    
这些命令还可以和数字继续组合，比如`y3w`复制3个单词， `2yy`复制三个单词。  
还可以用来复制配对的括号的内容， 比如`yi{`复制大括号`{`中的内容，`ya{`连括号一起复制。    

## 寄存器的进阶用法

### 多种多样的寄存器

使用`:reg`查看寄存器，Vim的寄存器可以让复制粘贴玩出更多的花样， 下面介绍这些寄存器。  
Vim的0-9寄存器是普通的寄存器，y命令默认会放到0，然后每次y会把最新的放到0，0放到1，依次保存结果。 如果你想粘贴哪个寄存器的值，在insert模式下`Ctrl+R 寄存器`， 比如粘贴寄存器3：`Ctrl+R 3`。

比较有用的是4个只读寄存器和搜索寄存器：

`.`  最近插入的文本  
`:`  最近执行的命令  
`%`  存放当前文件名（相对）  
`#`  交替文件（这个好像没啥用, 就是你在当前terminal打开的上一个文件的文件名）  
`/` 搜索寄存器，可以设置默认的搜索内容。比如`:let @/ = "package main"`, 下次搜索的时候只要输入`/`， 会自动查找`package main`这段文本

### 一些技巧

`xp` 交换两个单词，x将一个单词剪切到寄存器，p复制（利用错位交换） 
（交换两个单词建议用abloish插件）  

如果你删了东西后悔了可以去小删除寄存器里面找

### Vim剪切板和x11的剪切板交互

在设置中`set clipboard^=unnamed,unnamedplus`，开启系统的剪切板，就会多出来两个寄存器`*`和`+`。一般`*`是PRIMARY，`+`是CLIPBOARD。（开系统剪切板功能你可能需要gvim或者neovim，vim这个包可能不支持）  

当然这么干会把匿名寄存器“”的值和系统的CLIPBOARD复制粘贴绑定，不过比起少打几个`+`， 可以接受。
（并没有绑定PRIMARY，`*p`会粘贴CLIPBOARD的内容，PRIMARY直接鼠标中键粘贴就可以了。所以复制同步的过程并不会在每次鼠标选中文本的时候发生，只有在手动`Ctrl+Shitf+C`的时候才有，可以接受）

## 链接

<https://wiki.archlinux.org/title/Clipboard>   
[X11: How does “the” clipboard work?](https://www.uninformativ.de/blog/postings/2017-04-02/0/POSTING-en.html)   
[Vim registers: The basics and beyond](https://www.brianstorti.com/vim-registers/)  

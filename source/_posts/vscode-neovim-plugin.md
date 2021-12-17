---
layout: posts
title: 将VSCode作为Neovim的GUI客户端——VSCode Neovim插件配置
abbrlink: 'vscode-neovim-setting'
date: 2021-10-30 0:0:0
tags:
 - Neovim
 - Vim
 - VSCode
categories:
  - 不务正业系列
---
本来不报希望的试了试这套不伦不类的缝合怪配置，结果缝合出了很好的效果，可以拿这套配置在VSCode下养老了。
<!-- more -->

VSCodevim这个插件和原生Vim的体验差距很大，Vim的很多操作都不支持，而且大文件会特别卡。VSCode Neovim这个插件是在Insert模式几乎和正常的VSCode编辑一样，在Normal模式有满血的Vim体验（不像VSCodevim那样是体验是残血的，只是一个Vim键位和操作的模拟器）

这个插件直接将Neovim作为后端，在Normal模式下由Neovim控制（直接将内容缓存发往Neovim处理），可以使用Vim的各种键位操作   
在Insert模式下，操作和渲染由VSCode处理，编辑体验几乎和VSCode完全一致。色彩高亮和代码补全等都由VSCode实现（所以不要装Neovim的LSP等插件，没用，而且可能影响速度）当你保存的时候更改会从VSCode同步到Neovim。  

VSCode在编辑模式下的舒服体验和插件、Vim在Normal模式下的操作，同时得到了保留，而且性能和速度还可以接受。

与其说这是VSCode的一个Vim键位插件，不如说是将VSCode直接改造成了Neovim的一个GUI客户端。

## 安装

VSCode插件市场装VSCode Neovim，Archlinux安装neovim`sudo pacman -S neovim`
然后在设置里面填上路径

```json
    "vscode-neovim.neovimExecutablePaths.linux": "/usr/bin/nvim",
```

就可以用了

## 中文输入法和Neovim插件Normal模式下冲突

VSCode的Neovim插件在Normal模式下，如果fcitx5等输入法处于中文模式，那么输入的字符会被输入法全部捕获（就像平时在VSCode里打字一样）

ArchWiki 给出了下面这种方法，试了一下速度比较慢

```vimrc
autocmd InsertLeave * :silent !fcitx5-remote -c " 退出插入模式时禁用输入法
autocmd BufCreate *  :silent !fcitx5-remote -c " 创建 Buf 时禁用输入法
autocmd BufEnter *  :silent !fcitx5-remote -c " 进入 Buf 时禁用输入法
autocmd BufLeave *  :silent !fcitx5-remote -c " 离开 Buf 时禁用输入法
```

比较快的方法是装插件，比如依云的fcitx.vim插件，但是这个插件在neovim下面好像没有效果。

本来想自己移植一个neovim的插件的，但是看了一下已经有人干了：[fcitx.nvim](https://github.com/h-hg/fcitx.nvim.git)

我用的vim-plug管理插件，放到`~/.config/nvim/init.vim`下面就可以了

```vim
call plug#begin()
Plug 'https://github.com/h-hg/fcitx.nvim.git'
call plug#end()
```

## 一些按键绑定设置

首先我不要hjkl这几个上下左右键（根本无法接受，我的肌肉记忆明明是键盘上上下左右四个经典方向键，玩空洞骑士和Ballance用的极其熟练，其次是WASD这个4399键位，其次是鼠标，我鼠标可熟练了）  
我知道hjkl移动距离短而且移动后方便按距离近的i键进入Insert模式，但是我的肌肉记忆是上下左右，你再给我十年我也改不过来。

然后我将Casplock映射为ESC，这个万年不用的键放在这么重要的位置还有一个比Tab键还大的键帽，简直浪费。编辑`～/Xmodmap`修改键盘映射就可以了。

或者如果喜欢也可以将`jj`（连按两次）绑定成ESC`inoremap jj <Esc>`^`

然后是VSCode里面正常的`Ctrl+F`当前搜索，这个我们找回来（`Ctrl+F`在Vim里是向上翻页，我完全用不到，我用PgDn和PgUp）

然后是VSCode的`Ctrl+B`展开/收起侧边栏，这个我也留着VSCode的设置。

编辑`~/.config/Code/User/keybindings.json`

```json
{
        "key": "ctrl+f",
        "command": "-vscode-neovim.escape",
    },
    {
        "key": "ctrl+b",
        "command": "-vscode-neovim.escape",
    },
    {
        "key": "ctrl+a",
        "command": "-vscode-neovim.escape",
    },
```

复制和粘贴我大部分时间直接用linux下x默认的鼠标点击和中键。但是全选等操作的时候我还是习惯`Ctrl-A`加`Ctrl-C`

偶尔使用`Ctrl-C`的人，把`Ctrl-C`和`Ctrl-A`找回来还是有必要的，偶尔在`insert`模式下`Ctrl-C`会进入normal模式，`Ctrl-A`会输入`。sd`，很烦。

所以我也直接在json设置里面排除了`Ctrl-A`。ps：如果你想使用mswin.vim那样在neovim里面绑定，那可能会报错`nvim_call_function: Vim(let):E684: list index out of range: 0`。注意在Neovim的扩展设置里面（Keyboard shortcuts）删掉`Ctrl+A`

然后在`settings.json`中设置

```json
    "vscode-neovim.useCtrlKeysForInsertMode": false,
    "vscode-neovim.useCtrlKeysForNormalMode": false,
```

这样就基本上把Ctrl的功能还给VSCode了。

## Neovim插件默认instert模式

这个样子平时就像正常使用VSCode一样，当用到normal mode的功能时才打开normal mode，可以少按很多次i

```vim
if has('nvim')
    autocmd TermOpen term://* startinsert
endif
```

## 后记

之前只在编辑小文件的时候临时用Vim, 这两年内Vim使用次数逐渐增多，Vim也逐渐熟练，一度产生过将主力编辑器从VSCode换成Vim的想法。VSCode在Linux平台上频繁内存泄漏、VSCode的渲染速度比Vim慢很多（尤其是打开大文件的时候），这些原因导致VSCode在某些时候的体验比Vim差了好多。

但是Vim我没有找到合适的GUI界面来打造一个对我而言比较舒服的编辑器，gvim在当年上嵌入式课的时候在Ubuntu下就试过一次了，感觉体验和

SpaceVim试过，装了一堆插件之后太卡，性能不是太好，性能表现几乎和VSCode不相上下，在我的超低配七年老电脑上失去了Vim流畅的优势，而且这一套界面的颜值和使用体验远不如VSCode。

Neovim的GUI界面有[goneovim](https://github.com/akiyosi/goneovim)和neovide，体验也不如VSCode。（go居然能拿来写Qt还写成这种样子，哇偶）

最舒服的Vim体验还是在Alacritty里面，GPU加速渲染速度很快，非常丝滑。

而且VSCode在我从18年开始使用它的三年内，肉眼可见的进步。未来的前景也很光明，微软掏钱养着也不用担心倒闭的问题，除非微软倒闭（真倒闭了那不是更好吗，还有这种好事，苏联笑话.jpg）  
现在我除了要写一万行以上的屎山项目才会打开Goland等全家桶IDE，编辑100行以下的配置文件等会打开vim。
剩下的绝大多数编辑都是用VSCode,用VSCode来写Go、C、Python、shell，用VSCode来写Markdown和LaTeX, 逐渐抛弃了typora和Word，也放弃了曾经很喜欢的sublime text（纯粹是因为好看和启动速度比vscode快一点）

## 链接

[从VSCode到Vim到……两个都用？](https://www.ahonn.me/blog/the-vim-guide-for-vs-code-users)   
https://github.com/daipeihust/im-select#installation Windows和Mac以及使用vscodevim插件用户可以用这个解决中文输入法冲突的问题  
https://ddadaal.me/articles/from-vscode-to-vim-to-both   
[Vim 和 Neovim 的前世今生](https://jdhao.github.io/2020/01/12/vim_nvim_history_development/)  
[vscode 集成 Neovim](https://www.jianshu.com/p/ac739c6ea541)  
[VSCode with embedded Neovim](https://www.youtube.com/watch?v=g4dXZ0RQWdw)  
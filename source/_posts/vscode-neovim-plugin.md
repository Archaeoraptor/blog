---
layout: posts
title: VSCode Neovim插件配置——将VSCode作为Neovim的GUI客户端
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

VSCode插件市场装VSCode Neovim，安装neovim`sudo pacman -S neovim`
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

```viml
call plug#begin()
Plug 'https://github.com/h-hg/fcitx.nvim.git'
call plug#end()
```

## 一些按键绑定设置

我将Capslock映射为ESC，这个万年不用的键放在这么重要的位置还有一个比Tab键还大的键帽，简直浪费。我用`interception-caps2esc`交换了ESC和Capslock，并且让原Caps键作为组合键的时候是Ctrl（这个是利用udev交换的）

这样还有一个好处是换到Caps的ESC键离和a（append）非常近，这样只要按Caps和a就可以切换normal和insert模式了。

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

更新：让VSCode的左侧文件侧栏（file explorer）能用jk进行上下浏览文件。
如果装了vscodevim插件会有这个功能。vscode默认没有这个设置选项，看起来官方也不打算加，我们借助插件the multi-command extension实现，`settings.json`加上

```json
"multiCommand.commands": [
  {
    "command": "multiCommand.navigateExplorerDownAndPreviewFile",
    "sequence": ["list.focusDown", "filesExplorer.openFilePreserveFocus"]
  },
  {
    "command": "multiCommand.navigateExplorerUpAndPreviewFile",
    "sequence": ["list.focusUp", "filesExplorer.openFilePreserveFocus"]
  }
]
```

`keybingdings.json`加上

```json
{
  "key": "down",
  "command": "multiCommand.navigateExplorerDownAndPreviewFile",
  "when": "explorerViewletVisible && filesExplorerFocus && !explorerResourceIsRoot && !explorerResourceReadonly && !inputFocus"
},
{
  "key": "up",
  "command": "multiCommand.navigateExplorerUpAndPreviewFile",
  "when": "explorerViewletVisible && filesExplorerFocus && !explorerResourceIsRoot && !explorerResourceReadonly && !inputFocus"
}
```

我是不喜欢用hjkl，我用上下左右四个方向键，玩空洞骑士和蔚蓝上下左右的方向键已经用习惯了。而且单独的方向键有个好处是可以在 insert mode 直接进行移动，不用进入  normal mode 再按hjkl。

大部分标准键盘的上下左右是符合操作直觉的，上就在上面下就在下面。hjkl那么扭曲我不想去用。不过有些阴间键盘，尤其是笔记本.......


## Neovim插件默认instert模式

这个样子平时就像正常使用VSCode一样，当用到normal mode的功能时才打开normal mode，可以少按很多次i

```vim
if has('nvim')
    autocmd TermOpen term://* startinsert
endif
```

## 在使用VSCode时选择性启用Neovim插件

Vim/Neovim的插件和VSCode的插件有功能重叠，我目前各选了一部分。目前我的LSP插件用VSCode的插件，其他像Markdown插件、LaTeX插件和各种杂七杂八不常用的功能也都是用的VSCode插件。我装了EasyMotion的替代品MetaJump（VSCode插件），用于进行跳转。Neovim插件我保留了`vim-surround`等键盘操作的插件。

但是很多时候我要在终端用Neovim的时候要启用一些插件，使用VSCode的时候要禁用Neovim部分插件。

推荐按照官方文档在使用VSCode时禁用Neovim的自动补全、LSP类、语法高亮等插件

>You don't need any code, highlighting, completion, lsp plugins as well any plugins that spawn windows/buffers (nerdtree and similar), fuzzy-finders, etc.

具体参考[官方文档](https://github.com/vscode-neovim/vscode-neovim#conditional-initvim), 我用的vim-plug可以这样设置

```
" inside plug#begin:
" 在使用neovim的时候启用vim-easymotion插件
Plug 'easymotion/vim-easymotion', Cond(!exists('g:vscode'))
" 在vscode-neovim插件模式下启用另一个插件
Plug 'asvetliakov/vim-easymotion', Cond(exists('g:vscode'), { 'as': 'vsc-easymotion' })
```

禁用neovim的语法高亮：

```lua
if vim.g.vscode then
  vim.cmd.colorscheme = ""
```

不过后来我的画风逐渐变成了这样：
Neovim专心给vscode当backend用，terminal里面用vim，这样就不用管插件何时启用了。

如果不需要Neovim的任何插件和语法高亮等功能，可以直接让Neovim以clean mode启动，这样会减少很多出问题的几率。

```json
"vscode-neovim.neovimClean": true
```

## 后记

之前只在编辑小文件的时候临时用Vim, 这两年内Vim使用次数逐渐增多，Vim也逐渐熟练，一度产生过将主力编辑器从VSCode换成Vim的想法。VSCode在Linux平台上频繁内存泄漏、VSCode的渲染速度比Vim慢很多（尤其是打开大文件的时候），这些原因导致VSCode在某些时候的体验比Vim差了好多。

但是Vim我没有找到合适的GUI界面来打造一个对我而言比较舒服的编辑器，gvim在当年上嵌入式课的时候在Ubuntu下就试过一次了，感觉体验不是很好。

SpaceVim试过，装了一堆插件之后太卡，性能不是太好，性能表现几乎和VSCode不相上下，在我的超低配七年老电脑上失去了Vim流畅的优势，而且这一套界面的颜值和使用体验远不如VSCode。

Neovim的GUI界面有[goneovim](https://github.com/akiyosi/goneovim)和neovide，体验也不如VSCode。（go居然能拿来写Qt还写成这种样子，哇偶）

最舒服的Vim体验还是在Alacritty里面，GPU加速渲染速度很快，非常丝滑。

而且VSCode在我从18年开始使用它的三年内，肉眼可见的进步。未来的前景也很光明，微软掏钱养着也不用担心倒闭的问题，除非微软倒闭（真倒闭了那不是更好吗，还有这种好事，苏联笑话.jpg）  
现在我除了要写一万行以上的屎山项目才会打开Goland等全家桶IDE，编辑100行以下的配置文件等会打开vim。
剩下的绝大多数编辑都是用VSCode,用VSCode来写Go、C、Python、shell，用VSCode来写Markdown和LaTeX, 逐渐抛弃了typora和Word，也放弃了曾经很喜欢的sublime text（但是这个好看、比VSCode丝滑，VSCode的渲染和响应速度没有那种丝滑的感觉）

### 2022.4更新

用VSCode Neovim插件半年了，多多少少有点小bug，偶尔出现一些光标不灵和Neovim后端没有启动的小问题。大部分时候reload一下就好了。大多数时候编辑小文件的体验都是alacritty中打开vim更好，打开速度飞快而且打字延迟低、渲染都比VSCode舒服。但是vim/neovim要想配置出vscode/sublime写大一点的项目的工作界面来比较麻烦，所以大多数时间还在用VSCode，没啥迁移的动力。

用久了发现VSCode的很多图形界面实在没有必要，很多边框和按钮基本用不到还占地方那个。比如最上面的边框（用i3wm等wm不要边框就好了），然后是那个`Menu Bar`，用鼠标点`Menu Bar`是很浪费时间的，建议隐藏了。需要进行什么操作建议`F1`搜索，比拿鼠标点点点舒服。

然后比较讨厌的的是下方terminal和编辑界面中间宽大的框，左侧边栏（Side Bar）也特别宽（这个间距还不能调，只能`Ctrl+“-“`进行zoom缩放变相调小）。这个特别难受，当时我刚用VSCode的时候我就觉得VSCode的界面比sublime和jetbrains家的IDE浪费空间好多。

这个极为浪费空间的UI布局就像新版firefox的proton和新版gnome的移动平板风格，太讨厌了。大概就是这个对比的感觉：[Firefox-UI-Fix](https://github.com/black7375/Firefox-UI-Fix)  

我最早一直看VSCode的界面不顺眼，换上atom或者sublime的主题也不行。后来想了很久这是为什么，最后发现就是这个布局太不紧凑了，太浪费空间了。  
我心目中好的界面是像大部分QT应用那样的，虽然第一眼颜值没有electron高，但是紧凑高效。比如WPS、Photoshop、Krita这样的，没有特别宽的没用边框出来浪费空间。AutoCAD、Altium Designer和JB家的编辑器也比较紧凑。就连VSCode的竞品atom和sublime也远比VSCode紧凑。

不过我在找VSCode紧凑布局的办法的时候，发现VSCode有像Vim差不多的 Zen Mode，`Ctrl+k z`切换为禅意模式，临时进入一个全屏的清爽的编辑界面，默认只有一个居中的编辑界面。有点像typora的全屏+专注模式，感觉还行。

## 2022.5更新

建议添加，可以缓解卡顿的问题，详见[Extensions using the "type" command (for ex. Vim) have poor performance due to being single-threaded with other extensions](https://github.com/microsoft/vscode/issues/75627) 和 [Overriding the default 'type' command and then calling the default 'type' command results in significantly slower execution time](https://github.com/microsoft/vscode/issues/65876)

```json
"extensions.experimental.affinity": {
        "asvetliakov.vscode-neovim": 1
    },
```

## 链接

[从VSCode到Vim到……两个都用？](https://www.ahonn.me/blog/the-vim-guide-for-vs-code-users)   
https://github.com/daipeihust/im-select#installation Windows和Mac以及使用vscodevim插件用户可以用这个解决中文输入法冲突的问题  
https://ddadaal.me/articles/from-vscode-to-vim-to-both   
[Vim 和 Neovim 的前世今生](https://jdhao.github.io/2020/01/12/vim_nvim_history_development/)  
[vscode 集成 Neovim](https://www.jianshu.com/p/ac739c6ea541)  
[VSCode with embedded Neovim](https://www.youtube.com/watch?v=g4dXZ0RQWdw)  
[在 neovim 中使用 Lua](https://github.com/glepnir/nvim-lua-guide-zh)  
[vscode-neovim](https://github.com/vscode-neovim/vscode-neovim)  
[LunarVim](https://www.lunarvim.org/) 这个项目不错，从这学了不少Neovim的配置  
[LunarVim Core plugins](https://www.lunarvim.org/plugins/#configuration) 这里面的插件质量都还不错，挑几个拿过来用挺好的  
[Neovim IDE from Scratch - Introduction (100% lua config)
](https://www.youtube.com/watch?v=ctH-a-1eUME) LunarVim作者的教程  
[Neovim from Scratch](https://github.com/LunarVim/Neovim-from-scratch)  
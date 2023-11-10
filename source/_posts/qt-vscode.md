---
title: VSCode配置QT开发环境
tags:
  - Qt
abbrlink: qt-vscode
hide: true
date: 2023-10-24 10:01:19
---

TLDR：用QTCreator新建一个项目，然后用VSCode打开。
<!-- more -->

试用了VSCode上面的几个QT插件，最后发现还是直接用QTCreator生成一个项目然后用VSCode打开最方便省心。

QTCreator自然功能很多，但是快捷键和操作没有VSCode用着顺手。我也不是很喜欢用IDE，有语法高亮、自动补全、定义跳转就行了。而且QTCreator的fakevim很不好用，比原生vim或者装了Neovim插件的VSCode差太多了。

而且QTCreator在我的电脑上一个最要命的问题是，自动补全太慢了，往往我单词都快要敲完了自动补全还没有跳出来。

除了编辑ui文件的时候用QTCreator打开以外，其他的体验都是VSCode或者vim更好一些。

```bash
sudo pacman -S qt5-base
sudo pacman -S qmake make cmake
```

然后用工具人QTCreator新建一个项目，用VSCode打开。

自动补全和跳转我采用clangd插件，需要生成`command_compile.json`。

qt5的qmake项目可以用bear生成`command_compile.json`，我是新建一个build目录并加`.gitignore`里，然后用bear生成，最后放到根目录里就行了。

```bash
sudo pacman -S bear
```

```bash
mkdir build
cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=1 .
qmake ..
bear -- make
cp compile_commands.json ../
```

然后在build目录中运行编译生成的可执行文件就好了`./YourQtProject`

然后还装了一个QT for python的插件用于语法高亮QSS。  
最后在QTCreator的编辑器设置自动同步外部更改。  

完
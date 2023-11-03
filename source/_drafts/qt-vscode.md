---
title: VSCode配置QT开发环境
date: 2023-10-24 10:01:19
tags:
- qt
abbrlink: 'qt-vscode'
hide: true
---
TLDR：用QTCreator新建一个项目，然后用VSCode打开。
<!-- more -->

QTCreator自然很好用，但是快捷键和操作没有VSCode用着顺手。我也不是很喜欢用IDE，反正只要有语法高亮、自动补全、跳转就差不多够了。

```bash
sudo pacman -S qt5-base
sudo pacman -S qmake make cmake
```

然后用QTCreator新建一个项目，用VSCode打开。

自动补全和跳转采用clangd插件，需要生成`command_compile.json`。

qt5的qmake项目可以用bear生成`command_compile.json`，我是新建一个build目录并加`.gitignore`里，然后用bear生成，最后放到根目录里就行了。

```bash
mkdir build
cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=1 .
qmake ..
bear -- make
cp compile_commands.json ../
```

然后在build目录中运行编译生成的可执行文件就好了`./YourQtProject`

完（逃
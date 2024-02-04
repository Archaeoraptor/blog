---
title: VSCode C/C++ 开发环境和调试配置：Clangd+Codelldb
date: 2022-03-20 20:02:25
tags:
- VSCode
- clangd
abbrlink: 'vscode-c-and-cpp-develop-and-debug-setting'
---
卸了cpptools和C++ Intelligence吧，来试试clangd或者ccls
<!-- more -->

## 太长不看版

别用微软官方的那个C/C++扩展（cpptools）和 C++ intelligence 扩展，用 clangd 和 CodeLLDB。
**全文完。我们下次再见**

## 安装和配置

寻找保姆级教程、习惯鼠标操作、想将VSCode打造成一个C/C++ IDE的读者建议立刻退出并寻找其他教程。以下操作以Archlinux为例，如果你是Windos用户请去用尊贵的宇宙第一IDE Visual Studio 或者通过WSL和VSCode Remote获得一个linux的开发环境。以下假定读者有一定的基础和使用搜索引擎查资料的能力。**这不是一个保姆级教程、详细教程**。  

配置环境本应该是一件很简单的事情，然而M$的官方文档在推自己的cpptools塞私货，大量配置教程不是太啰嗦就是教你用cpptools，已经造成了信息污染。于是有了这篇clangd和Codelldb安利文。

事实上这是很简单的，你只需要干这些事：

### 安装VSCode

通常一条命令就可以了，已安装用户请略过

```bash
# 安装code-oss版本，和插件市场
yay -S code code-marketplace
# 如果你想装闭源的VSCode
yay -S visual-studio-code-bin 
```

### 安装编译器

想用gcc就用gcc，无特殊需求系统自带的gcc已经足矣（有特殊版本和交叉编译请自行安装对应版本）。想用clang就装clang。

```bash
yay -S llvm lld lldb clang
```

### 安装插件

点击安装就可以

[clangd插件](https://marketplace.visualstudio.com/items?itemName=llvm-vs-code-extensions.vscode-clangd)  用于高亮、自动补全、跳转  
[CodeLLDB插件](https://marketplace.visualstudio.com/items?itemName=vadimcn.vscode-lldb)  如果你需要图形界面debug

ps: 如果你用闭源的VSCode，那就直接在M$的插件市场装。Code-OSS版本可以修改`product.json`使用插件市场。如果你使用或者VSCodium版本，请从OpenVSX安装[clangd插件](https://open-vsx.org/extension/llvm-vs-code-extensions/vscode-clangd)（或者手动下载）

### 安装clangd LSP

然后按`F1`，选`clangd:Download Language Server`, 下载clangd LSP （如果你想用系统包管理器装clangd也可以, Archlinux在`llvm`这个包里，Debian等发行版有单独的`clangd`包）

### clangd插件主要功能和配置

切换c/cpp文件和`.h`头文件：Alt+O

clangd: Open Type Hierarchy 显示类的继承关系，我一般不怎么用类，除非写Qt。比较鸡肋的功能。

clangd-tidy系列检查工具：



## Debug和运行

编译运行这个就没必要装插件了吧，单个文件我使用gcc/g++命令或者clang/clang++命令，编译并运行。一个项目那就写个Makefile或者cmake之类的东西呗。

比如一个简单的cmake：

```cmake
cmake_minimum_required (VERSION 3.28.1)
project (Transformation)

find_package(Eigen3 REQUIRED)
include_directories(EIGEN3_INCLUDE_DIR)

add_executable (Transformation main.cpp)
```

如果vscode装了cmake插件可以F7执行build

### 使用gdb和lldb命令Debug

如果你需要Debug，在终端使用gdb和lldb即可， over～

### 使用VSCode提供的图形界面Debugger

哦，你要图形界面啊，安装Codelldb插件。`Ctrl+Shift+D`点左边栏的debug按钮，然后应该会自动生成一个`launch.json`，然后自行配置你的debug命令、参数和环境变量，参考：[VSCode Debugging](https://code.visualstudio.com/docs/editor/debugging)

`tasks.json`如下，我以c语言为例，设置输出到`build`文件夹下同名`.out`文件，有需要请自行修改。

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "Compile",
            "type": "process",
            "command": "clang",
            "args": [
                "${file}",
                "-g",
                "-o",
                "${fileDirname}/build/${fileBasenameNoExtension}.out", // 输出到build文件夹下
                "-Wall",
                "-std=c11",
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "options": {
                "cwd": "${fileDirname}"
            },
        }
    ]
}
```

`launch.json`如下，注意`preLaunchTask`要先执行`task.json`的label为`Compile`的任务

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "LLDB Debug",
            "type": "lldb",
            "request": "launch",
            "program": "${fileDirname}/build/${fileBasenameNoExtension}.out",
            "cwd": "${fileDirname}",
            "preLaunchTask": "Compile"
        }
    ]
}
```

好了，这样就可以了。如果是cmake项目，不需要tasks.json，大概要写成这样

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "lldb",
            "request": "launch",
            "name": "Debug",
            "program": "${command:cmake.launchTargetPath}",
            "args": [],
            "cwd": "${workspaceFolder}"
        }
    ]
}
```

然后按F5调试

## 其他的配置

### 调试时使用命令操作

按F5进行调试，此时按`Ctrl+Shift+Y`调出Debug Console REPL就可以用gdb/lldb的命令进行调试了。这样就又回到了熟悉的gdb/lldb的命令，就可以不需要鼠标点点点进行调试了，一些Codelldb不支持显示在图形界面的调试功能也可以用了。

### 每次调试都多产生一个Debug Console

给`launch.json`加上

```json
            //args的这个设置是为了防止每次Debug都会多弹出一个Debug Console
            "args": [   
                "&&",
                "exit"
            ],
```

## 随便说说

### 关于C/C++的LSP其他选择

目前用的比较多的C/C++的LSP是ccls和clangd，各有优劣。我的体验是ccls没有clangd的补全和错误提示那么舒服，就一直在用clangd，有兴趣可以试试ccls。
~~反正这两个都比cpptools好用就是了~~

### 关于VSCode和插件

本来是不想写这个的。但是看到你们官方文档闭口不提clangd，网上各路教程也都在教大家用cpptools和C++ intelligence这两个插件，我蚌埠住了。昨天一个小学弟配vscode的c/c++环境，折腾了一整天，然后跟我吐槽vscode又慢又卡，我心里暗叫不好，这不会是又用了cpptools和C++ intelligence这两个插件吧。我问小学弟为什么不用clangd，他表示官方文档也没说啊，他明明按照官方文档来的。好吧，这也不怪小学弟，连官方文档都在叫你用M$那两个又吃内存又难用的插件。Google能搜到的也都在让你装这个。甚至你从EXTENSOINS的搜索栏里面搜c或者c++，前两个插件就是这两个，至于clangd，已经排在无数个名字里有c/c++插件的后面找不着了。

**我的操作逐渐转为快捷键和terminal，图形界面主要用来展示而不是用来点点点完成操作**。我主要是将VSCode作为一个neovim/gdb/lldb的展示前端来用，[Neovim插件配置见上一篇博客](https://zhangjk98.xyz/vscode-neovim-setting/)。目前用习惯了VSCode还真没动力换别的，几次试图切到纯Neovim最后又滚回了VSCode+Neovim插件。  

VSCode正在变得臃肿，而且打字延迟比Sublime和Neovim高。VSCode丝滑的感觉远不如Sublime Text，也不如高配电脑上的Jetbrains家的全家桶。我比较喜欢VSCode一个是LSP，一个是没有太多捆绑功能+丰富的插件生态和开箱即用。但是最近这两年总感觉它不再想好好做一个编辑器了，总想抢IDE的活，可能是风头压过atom和sublime之后也失去了方向。单纯作为一个文本编辑器，VSCode的打磨和细节体验和Vim、Sublime都有还有不小的差距，但是现在眼看着要向一个IDE的方向狂奔了。

VSCode的插件良莠不齐，而VSCode本体也逐渐和我想要的编辑器差距越来越大。但是毕竟用习惯了。现在很多网上的教程都是把VSCode配置成IDE，装了一大堆插件，最后又慢又卡。大多数复杂的功能学习所用的时间很有可能赶不上学会以后帮你节省的时间，尤其是点点点的图形界面，每次更新按钮换位置又要从头找起。就算学也学一点一些不怎么变的快捷键和Vim操作，那些在肌肉记忆可以一直用下去。

我觉得如果VSCode你要用的开心，**不要当IDE用，就当一个编辑器用**。要有一种又不是不能用的**摆烂**心态，不要刻意去配置。简单配置到**能用就行**。

## 链接

<https://open-vsx.org/> M$的VSCode插件市场的开源替代
<https://clangd.llvm.org> clangd项目

<https://code.visualstudio.com/docs/editor/debugging>
<https://code.visualstudio.com/docs/introvideos/debugging>

## 番外：linux源码阅读环境配置

一般看这种源码几乎都是Vim下用gtags或者在vscode下用 gnu global 插件，不过clangd配置完成后有语义分析，体验更好一点（虽然clangd会对一些注释和宏报Warning）。  

看这种大项目的代码，首先不能卡，然后要有语法高亮和跳转。然后各种头文件和宏不要一堆飘红报错。我之前一直以为VSCode打开这种特别大的项目的表现不堪大任，后来发现只是插件的问题，clangd表现相当不错。 

下载kernel的源码，如果只是阅读的话推荐只下载当前版本的，不要下整个 git repo，vscode打开repo会比打开普通文件夹要吃内存。

```bash
wget https://mirrors.tuna.tsinghua.edu.cn/kernel/v5.x/linux-5.16.15.tar.xz
tar xf linux-5.16.15.tar.xz
cd linux-5.16.15
```

我们需要生成`compile_commands.json`，这个可以用bear或者`scripts/clang-tools/gen_compile_commands.py`这个自带的脚本。不过这两种方法都需要你编译一遍内核。

```bash
zcat /proc/config.gz > .config  # 复制本机的config作为编译的config，有需要请自行定制
bear -- make -j 12  # 生成compile_commands.json

## 或者用提供的python脚本生成
make    # 编译
python scripts/clang-tools/gen_compile_commands.py
```

编译要等一段时间，我用5600G编译大概40分钟。然后clangd会自动开始indexing，我这里有两万多个index，indexing大概花了20多分钟。到这里头文件和自动跳转都正常了，此时浏览代码的时候vsc内存占用占用大概1-2G左右。
我还加了这几个参数: 采用gnu89标准（新版的linux 5.18 要升到c11了，如果你是master，换成`-std=gnu11`）、将预编译头文件放到内存里、关闭 clangd format （这个挺吃资源的，就看个源码不需要自动format）

```json
{
    "clangd.arguments": [
        "-std=gnu89",
        "--pch-storage=memory",
        "-j=12",
    ],
    "editor.formatOnSave": false
}
```

由于这里只是阅读源码，所以 clangd-tidy 和全局自动补全这些参数（"--clang-tidy","--all-scopes-completion",）我都没有启用。
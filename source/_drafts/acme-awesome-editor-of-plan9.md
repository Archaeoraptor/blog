---
title: 沧海拾遗二，Acme：Plan9失落的编辑器
date: 2022-03-04 22:03:33
tags:
- plan9
- acme
abbrlink: 'acme-awesome-editor-of-plan9'
---
这是这个系列第二篇
<!-- more -->

## 管道

我对图形界面和编辑器的印象是面向对象那一套和鼠标点点点，改变了我的

## 安装和使用

acme的操作习惯和我们现在熟悉的大多数编辑器是不太一样的，如果只是玩玩

## 链接

https://github.com/mkmik/awesome-acme
https://9fans.github.io/plan9port/man/man1/acme.html
https://news.ycombinator.com/item?id=23779894
https://news.ycombinator.com/item?id=4533156


数数看刚好七字, [2022/2/18 20:48]
https://www.youtube.com/watch?v=dP1xVpMPn8M

数数看刚好七字, [2022/2/18 20:49]
看看这玩意，Go 老祖们的东西都很强

phn51, [2022/2/18 21:03]
[In reply to 数数看刚好七字]
这玩意太过硬核，连高亮和自动补全都没有，这直接决定了go标准库变量的命名采用了谭浩强风格。

数数看刚好七字, [2022/2/18 21:03]
[In reply to phn51]
问题是那个东西确实很强，终端都可以cut-paste

数数看刚好七字, [2022/2/18 21:04]
任何文本都是链接，任何选区都可以是命令

phn51, [2022/2/18 21:07]
[ Photo ]
简单p个图

数数看刚好七字, [2022/2/18 21:08]
我想如果这玩意把lsp功能加上，岂不是相当于 browser+terminal+file+exploer+ide

数数看刚好七字, [2022/2/18 21:09]
我从大二就想过，为啥terminal也相当于个editor。但是却要区分它俩，必须分开用

数数看刚好七字, [2022/2/18 21:11]
[In reply to phn51]
是不是本来是vscode?

phn51, [2022/2/18 21:11]
没，这个图比vscode流行早

phn51, [2022/2/18 21:11]
是 nano

phn51, [2022/2/18 21:12]
nano的key绑定是模仿的emaccs所以中间的头才会斜眼看它

phn51, [2022/2/18 21:12]
我这边p成acme，也是用emacs的功能回应你说的“任何文本都是链接，任何选区都可以是命令”

数数看刚好七字, [2022/2/18 21:14]
[In reply to phn51]
可以一边写文本，一边执行命令，而且同时被另一个 acme 编辑吗？

数数看刚好七字, [2022/2/18 21:14]
acme进程本身也是一组文件，可以被另一个acme写入

数数看刚好七字, [2022/2/18 21:14]
进程是文件，文件也是进程

phn51, [2022/2/18 21:15]
emacs 也肯定能做到类似功能，不必太纠结这个设计啊。

phn51, [2022/2/18 21:16]
我找个文章你看下

phn51, [2022/2/18 21:17]
你也说是9p了啊，我是说需求是需求，设计是设计。设计是手段，不是目的。

数数看刚好七字, [2022/2/18 21:17]
[ Photo ]
他把 acme 的 body 部分 cat 出来了，而且还能echo进去

phn51, [2022/2/18 21:17]
https://twitter.com/ruanyf/status/1123545858828525568

phn51, [2022/2/18 21:18]
这个故事表达了我的观点。

数数看刚好七字, [2022/2/18 21:18]
不过我还是继续用 vscode，没有自动补全用不了

数数看刚好七字, [2022/2/18 21:18]
要是 lsp 和语法高亮也整合进了这玩意就好了，我肯定用

phn51, [2022/2/18 21:19]
没必要因为他的设计着迷的。。。

Alex Channel, [2022/2/18 21:20]
[In reply to phn51]
哈哈哈!!!VIM 布道的还没有换呢

数数看刚好七字, [2022/2/18 21:20]
确实神奇啊，连toolbar都可以编辑，能自己加command

数数看刚好七字, [2022/2/18 21:23]
学go真是好投资，我也是看go帖子时发现这个视频
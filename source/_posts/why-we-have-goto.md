---
title: 我们为什么需要goto
date: 2022-01-07 19:08:03
tags:
- goto
- go
- coruntime
abbrlink: 'why-we-have-goto'
---
有一天，上大一的你在学C语言，老师抱着谭浩强的红书，将goto批判了一番，同学们，可不要用goto啊！ 
**But, No velociraptors attacked me.**  
<!-- more -->

>后来，G·加科皮尼和C·波姆从理论上证明了：任何程序都可以用顺序、分支和重复结构表示出来。这个结论表明，从高级程序语言中去掉GOTO语句并不影响高级程序语言的编程能力，而且编写的程序的结构更加清晰。

你对这一番话深信不疑，时间飞逝。有一天，你学了汇编，看着`jmp`指令跳来跳去怎么看怎么眼熟。  
有一天，你学完嵌入式的课想写一点驱动，一看别人代码里面c处理异常，怎么满屏的goto啊   
有一天，你在给一个骗钱项目糊屎山，一大堆乱七八杂的逻辑写了七层循环，突然有个需求你需要从第2层跳到第6层去，你开始一层一层地往外跳。“要是能直接跳到第6层就好了”，你心想，“粪土之墙不可杇也”。写个函数封装一下吧，这样就可以return跳出去了。写着写着你发现，函数这么多参数，传来传去的更乱了……

后来，你在altium designer上画着PCB，正在为双层板走线焦头烂额的你总感觉这飞来飞去的走线有点眼熟，却又想不起来
后来，你看到别人在夸赞unix管道，管道传来传去的，似乎让你想起了什么
后来，你知道了`set jump`和`long jump`，这又是另外一个故事了……
后来，你闲着没事干决定去翻一翻内核源码，哇，怎么好多goto哎……
于是你决定看看大喷子Linus是怎么说的：

>"No, you've been brainwashed by CS people who thought that Niklaus Wirth actually knew what he was talking about. He didn't. He doesn't have a frigging clue."

>"I think goto's are fine, and they are often more readable than large amounts of indentation."

>"Of course, in stupid languages like Pascal, where labels cannot be descriptive, goto's can be bad."

## 咳咳

我第一次接触goto不是在大一学c语言的时候，而是在高一的微机课上，当时还要带鞋套。当时用的是VB，老师教我们写一些汉诺塔之类的小东西。已经记不得VB是怎么写的了，只记得当时窗外阳光明媚，微机课的时光很快乐。老师也很和蔼

不要用goto这个观点可能是从Dij

`不要用goto`这话让c语言老师来说是不太合适的，毕竟c又没有像RAII那样的东西，也没有javascript那种锚点让你跳到指定循环

goto在c里面一般用在释放资源，处理异常，还有跳出循环

而且，C里面的goto远远不像早期BASIC那么危险

## 协程

我没跑题。协程的实现就是setjump和longjump跳过来跳回去。

## 状态机

你有没有觉得用c语言这样的东西写状态机好麻烦，明明学数字电路的时候状态机没这么麻烦的。
对啊，for,while,switch和if这样的控制流去写状态机就是很麻烦，这件事goto这样跳来跳去的东西最适合干了。而且当你return来return去的时候，你可能自己又发明了一遍goto。

一开始推荐用循环等结构化编程来代替goto

## 链接

[GOTO still considered harmful? [closed]](https://stackoverflow.com/questions/46586/goto-still-considered-harmful) stackoverflow的讨论  
[Why does Go have a "goto" statement](https://stackoverflow.com/questions/11064981/why-does-go-have-a-goto-statement)  
[Structured Programming with go to Statements](https://pic.plover.com/knuth-GOTO.pdf) 高德纳写的，那句“过早的优化是万恶之源”就是在这里出现的。这篇里面认可了一部分goto的用法，他自己也用了一些goto  
[GOTO still considered harmful? [closed]]
[Linux: Using goto In Kernel Code](http://web.archive.org/web/20130521051957/http://kerneltrap.org/node/553/2131)  
为什么说 goto 是一种不好的用法？ - 大饼卷鸡蛋的回答 - 知乎
https://www.zhihu.com/question/20259336/answer/1781979621




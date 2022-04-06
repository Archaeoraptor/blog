---
title: 用go实现一个lisp方言
date: 2022-04-02 10:59:36
tags:
- lisp
- go
- interpreter
---
终于圆了一个心愿，照猫画虎整了个lisp方言出来。
<!-- more -->

## 缘起

很久之前就听说过SICP这本书，但是断断续续都没看完几章。后来研一的时候上水课不想听闲着没事看《The Little Schemer》，看了一大半，后来水课上完了，这本小册子也没再看了。  
再后来没事翻了翻《Write a Interpreter in Go》，是用go写一个叫Monkey语言的解释器，跟着这本书照猫画虎抄了一个，也不是很明白。但是总感觉这个远不如之前所看到的lisp优雅


一直到现在，我都没有系统的学过编译原理，对编译器和虚拟机也不懂。单纯的野路子玩票性质，lisp的函数式编程

## 小结

用go实现一个解释器其实不是那么舒服，如果对lisp比较熟建议直接用racket或者chezscheme去做一个lisp方言。  

go有很多命令行


## 链接

[](https://www.abnerchou.me/BuildYourOwnLispCn)  最早看到lisp解释器的地方，这次写lisp解释器也是大部分参考了这个   

自己动手实现Lua 用go实现一个lua

https://github.com/karminski/pineapple/blob/main/README-zh-CN.md

https://github.com/ajlopez/Golin README里面的资料差不多够用了
[怎样写一个解释器](http://www.yinwang.org/blog-cn/2012/08/01/interpreter)  
[](https://github.com/kedebug/LispEx)  
[](https://v2ex.com/t/124125)  
[](http://kedebug.me/blog/lisp-in-go/)
[](https://github.com/LixvYang/Writing-a-Interpreter-in-Go-Translation)
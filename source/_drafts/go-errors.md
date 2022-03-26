---
title: go的错误处理和异常
date: 2021-05-06 10:33:43
tags:
- go
abbrlink: 'go-errors'
categories:
- Concurrency&Go
---
曾经在C++和Python中愉快的使用`try catch`，最近用了几个月Go之后，被Go里面占了行数一半的
<!-- more -->

```go
if err != nil { 
    return err
}
```

错误处理搞得浑身难受。但是转念一想好像之前用了挺久的C语言里面也没有`try catch`这个东西啊，也是`return`一个返回值，只有一次用了`setjump`。这才发现我之前都没怎么认真研究过错误和异常的处理，最近有空，看看这个东西。
<!-- more -->

## 错误

## 关于 try catch

以前只听过c++里的 try catch 是用`setjump`和`longjump`实现的

## 链接

[C语言中的异常处理机制](http://ibillxia.github.io/blog/2011/05/03/Exception-handling-mechanism-in-c/)
[Golang错误和异常处理的正确姿势](https://www.jianshu.com/p/f30da01eea97)  16年的老文

https://github.com/ossrs/srs setjump 和 longjump
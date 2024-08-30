---
title: 动态链接库中函数或变量重名问题
date: 2024-08-30 13:49:08
abbrlink: 'dynamic-lib-same-var'
tags:
- ELF
- gcc
- c
---
全局变量，坏
<!-- more -->

## 重名导致coredump

TLDR，这是同名类析构了多次导致的。

```cpp

```

## 两个不同的动态库变量重名


```bash
export LD_LIBRARY_PATH=./:$LD_LIBRARY_PATH 
```


## 两个静态（static）变量重名
  

## 延迟绑定

延迟绑定

PLT（Procedure Linkage Table）表和GOT（Global Offset Table）表用来延迟绑定（Lazy Binding）


GOT表是全局偏移表


在编译时使用-fPIC选项，
编译时使用-fPIE选项，


-Bsymbolic选项



http://wiki.c2.com/?GlobalVariablesAreBad
https://dzone.com/articles/why-static-bad-and-how-avoid
https://stackoverflow.com/questions/7026507/why-are-static-variables-considered-evil
https://stackoverflow.com/questions/1052168/thread-safe-static-variables-without-mutexing

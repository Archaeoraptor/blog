---
title: λ演算笔记
abbrlink: a10f
date: 2019-10-29 21:59:33
tags:
 - lambda
 - scheme
mathjax: true
categories:
 - 不务正业系列
hide: true
---
运算规则简单，非常适合parser实现
<!-- more -->

$\alpha$替换和$\beta$归约

## λ演算(λ-calculus)

$\lambda$演算的运算符

$\lambda$演算中的表达式可以视为一个函数，满足

- 是匿名的
- 只有一个输入

接下来是自然数和运算法则。$\labmda$演算中是没有自然数这一类型的，只有接收一个参数并返回一个值的函数




## 附

### lambda演算计算器

scheme等语言虽然已经比较像$\lambda$演算，但毕竟不是

<https://okmij.org/ftp/Computation/lambda-calc.html>  

### scheme环境配置

Arch下安装mit-sheme包就好了，SICP的练习环境。这个操作基本跟Emacs差不多，是SICP的作业环境。

```bash
yay -S mit-scheme
```

也可以用 Chez Scheme，语法略有不同

```bash
yay -S chez-scheme
```

运行直接

```shell
scheme < test.scm
```

就好了

我不习惯Emacs的快捷键，都是在vscode里写的，装一个vscode-scheme插件就好了。可以在code runner插件里配置一下，然后按快捷键运行

```json
    "code-runner.executorMap": {
        ".scm": "scheme < ",
        "scheme": "scheme < ",
        }
```

## 链接

<https://github.com/cisco/ChezScheme>  


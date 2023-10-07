---
title: λ演算&函数式编程
abbrlink: a10f
date: 2019-10-29 21:59:33
tags:
 - λ-calcus
 - js
katex: true
categories:
 - 不务正业系列
hide: true
---
等待填坑

<!-- more -->

## 摸鱼一时爽

函数式编程(Functional Programming)好像比面向对象抽象程度更高。但是之前折腾硬件的时候都没有用到过，可能是不太适合处理IO接口和操作时序吧。
之前在图书馆乱翻，看到一本七周七语言的书，翻到函数式，感觉还挺有意思，教研室的摸鱼生活就搞点这个吧，反正干正事是不可能干正事的
~~正在进JavaScript的坑，将就着用js来练练手吧。~~
javascript味不对，还是直接上了Lisp

## λ演算(λ-calculus)

- 是匿名的
- 只有一个输入

多输入的函数要转成多个只有一个输入的函数的嵌套调用，也就是柯里化（currying）

$\lambda$项

## 附

### 环境配置

Arch下安装mit-sheme包就好了，SICP的练习环境。这个操作基本跟Emacs差不多，是SICP的作业环境。

```bash
yay -S mit-scheme
```

不过我用的是这个Chez Scheme，前几年开源了的东西，速度很快

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

详见 https://github.com/cisco/ChezScheme

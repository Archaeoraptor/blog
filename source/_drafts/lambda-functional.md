---
title: λ演算&函数式编程
abbrlink: a10f
date: 2019-10-29 21:59:33
tags:
 - λ
 - lisp
katex: true
categories:
 - 不务正业系列
hide: true
---

<!-- more -->


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


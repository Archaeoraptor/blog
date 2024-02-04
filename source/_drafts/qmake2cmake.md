---
title: QMake转CMake
tags:
- qmake
- cmake
abbrlink: "qmake2cmake"
hide: true
---
已经被c++屎山的环境依赖和构建工具逼疯。
<!-- more -->

一般我要么手写makefile，要么直接用meson。但是恰饭总得碰一下qmake和cmake。。

qt6官方放弃了qmake转cmake了，提供了一个脚本可以直接转。

```bash
git clone "https://codereview.qt-project.org/qt/qmake2cmake"
```

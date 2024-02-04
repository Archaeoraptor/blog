---
title: Qt打包动态库并调用 
date: 2023-11-06 11:28:43
tags:
- Qt
abbrlink: 'qt-dll'
categories:
- 笔记
---
一般QT多以动态链接库为主，因为静态链接库会有一些版权的问题。静态库步骤和动态库基本相同，选择的时候改一点编译参数就可以了。
<!-- more -->

## QTCreator新建库项目

本文QT环境为5.14.2，QTCreator为11.0.2。

已经存在的QWdiget项目快速改为动态链接库项目并调用可直接跳到下一节

QTCreator新建项目：
file -> new project -> library ->c++ library，然后选shared library

然后类名随便取（这里以testDynamicLib为例），然后选择依赖core，gui和widget按需选，如果有额外的依赖，比如sql和xml等，在`.pro`文件里面添加即可。项目除了没有main.cpp，长得和QApplication几乎一样，除了没有main.cpp，多了一个`_global.h`文件。

然后写一个twosum测试dll调用是否成功。

```bash
/mnt/c/U/0/c/testDynamicLib master !2 ❯tree
.
├── testdynamiclib.cpp
├── testDynamicLib_global.h
├── testdynamiclib.h
├── testDynamicLib.pro
└── testDynamicLib.pro.user

1 directory, 5 files
```

testdynamiclib.h头文件

```cpp
#ifndef TESTDYNAMICLIB_H
#define TESTDYNAMICLIB_H

#include "testDynamicLib_global.h"

class TESTDYNAMICLIB_EXPORT TestDynamicLib
{
public:
    TestDynamicLib();

    int twosum(int a, int b);
};

#endif // TESTDYNAMICLIB_H
```

testDynamicLib.cpp文件添加

```cpp
int TestDynamicLib::twosum(int a, int b)
{
    return a + b;
}
```

然后build，会在编译目录生成一个.dll和.lib文件。

然后创建一个新项目，file->new project->Application(Qt)->Qt Widget Application。把刚才的dll和lib复制到当前

点击添加库->外部库->选刚才的.lib库和平台。

然后`.pro`文件中会多这么一行（刚才的添加库操作等价于.pro配置中添加了一行LIBS配置）

```config
win32:CONFIG(release, debug|release): LIBS += -L$$OUT_PWD/./release/ -ltestDynamicLib
```

然后就可以调用了：

```cpp
    TestDynamicLib testLib;
    qDebug() << testLib.twosum(16, 4) << endl;
```

如果有widget等图形界面，调用也是类似的

```cpp
    QHBoxLayout *layout = new QHBoxLayout();
    testDynamicLib *testLib = new testDynamicLib(ui->widget);
    testLib->show();
```

## 已有的Applicaiton项目改C++ Library项目

明白了QTCreator创建的动态库和Applicaiton项目的区别，我们可知直接修改原有的项目，能使其变为动态库。

比如我们要写一个dll，c++ Library项目不方便运行和调试，可以在开发的时候直接建一个普通的可执行的Qt项目，然后在完成后改为动态库。(假设项目叫YOURPROEJ)

首先删掉`main.cpp`，然后新建一个

```cpp
#ifndef YOURPROEJ_GLOBAL_H
#define YOURPROEJ_GLOBAL_H

#include <QtCore/qglobal.h>

#if defined(YOURPROEJ_LIBRARY)
#  define YOURPROEJ_EXPORT Q_DECL_EXPORT
#else
#  define YOURPROEJ_EXPORT Q_DECL_IMPORT
#endif

#endif // YOURPROEJ_GLOBAL_H
```

其中`Q_DECL_IMPORT`和`Q_DECL_EXPORT`两个宏作用类似于.def文件。

然后在`yourproj.h`中include一下

```cpp
#include "yourproj_global.h"
```

然后修改`.pro`文件，添加

```config
TEMPLATE = lib
DEFINES += YOURPROJ_LIBRARY
```

然后在动态库中修改类

```cpp
// class YourProj : public QMainWindow
// 改为
class YOURPROJ_EXPORT YourProj : public QMainWindow
```

然后编译，调用即可。


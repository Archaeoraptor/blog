---
title: 聊聊QT中的耗时任务和多线程
date: 2023-10-13 16:22:27
tags:
- QT
abbrlink: 'qt-multithread'
---
顺便聊聊对GUI的碎碎念，以及为什么GUI的多线程都那么扭曲
<!-- more -->


最近在拿QT糊一些垃圾业务代码，好在我这数据量不大，就没管性能的问题。但是前几周朋友问了我一个问题，QT怎么写多线程。她在写QT的时候遇到了一个问题，点击按钮开始批量处理几千个数据文件。于是主线程被阻塞了，于是所有界面按钮都卡了，GUI停止响应了。（因为GUI的刷新绘制操作和事件响应也在主线程）

问题：耗时操作会阻塞主线程。

缓解这个问题的方式似乎很简单，把耗时操作从主线程中独立出来就好了。

QT的多线程比较简单，提供了QThread类，还有信号槽可以用于线程通信。

QT的多线程主要有两种方式

第一种，继承QThread











看了QT的多线程操作，我怎么看怎么难受，再看一遍还是难受，问题似乎解决了，但是哪里不对劲。这跟我之前看unix管道、go的channel和协程、javascript的异步、erlang的进程和Actor模型的感受完全不一样，这些东西是流畅自洽的

问：GUI操作难道不能像读写数据等其他操作一样多线程吗？

QT不想让你这么干，绝大多数GUI框架也不想让你这么干：

>As mentioned, each program has one thread when it is started. This thread is called the "main thread" (also known as the "GUI thread" in Qt applications). The Qt GUI must run in this thread. All widgets and several related classes, for example QPixmap, don't work in secondary threads. A secondary thread is commonly referred to as a "worker thread" because it is used to offload processing work from the main thread.

from: <https://doc.qt.io/qt-5/thread-basics.html#gui-thread-and-worker-thread>


目前绝大多数UI操作都是单线程的，UI操作都是在主线程中的。网上通常的解释是子线程操作容易死锁。



我之前用linux桌面的时候试过不少wm（window manager）

其他UI中的多线程

javascript：

js是经典的单线程操作，然后有一个web worker


declative imperative sentence 声明式 命令式

这几天突然想到似乎声明式框架的思路是

## 链接

https://v2ex.com/t/505607


https://leobert-lan.github.io/Android/Idea/post_9.html
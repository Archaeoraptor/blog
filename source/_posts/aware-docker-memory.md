---
title: 又被Docker坑了（这次是内存
date: 2022-04-24 22:07:46
tags:
- Docker
abbrlink: 'aware-docker-memory'
---
好端端一个Docker container，怎么就内存占用过高被kill了呢
<!-- more -->

好吧这好像也不全算Docker的锅，罪魁祸首竟是dentry

## dentry还我内存

发现是dentry的锅

## Docker的一些替代品

说起来当时把那几台老的Ubuntu16.04和远古版本的

Docker看起来简单但是坑真的不算少，要用好Docker（很多人是图方便才来用Docker的）

如果只是日常使用为了装一些软件或依赖方便呢，装一个Archlinux显然更方便（这比搞懂Docker那一堆网络、layer、）

## 链接

https://www.ardanlabs.com/blog/2020/02/docker-images-part1-reducing-image-size.html

https://zhuanlan.zhihu.com/p/43133085  
https://blog.arstercz.com/centos-系统-slab-dentry-过高引起系统卡顿分析处理/  
https://hackernoon.com/another-reason-why-your-docker-containers-may-be-slow-d37207dec27f  
https://www.cnblogs.com/gaogao67/articles/15568378.html  
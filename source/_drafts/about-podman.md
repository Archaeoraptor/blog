---
title: podman二三事
date: 2020-11-26 17:32:20
tags:
abbrlink: 'about-podman'
---

以及其他的一点容器相关的东西

<!-- more -->

之前听早就听过podman的安利，一直用着Docker，后来用上了AUR仓库，装东西过于方便连Docker也不怎么用了。后来Docker和透明代理冲突才换的podman

## podman使用 VSCode Docker 扩展

## 一些报错和小问题

### 一些报错

```log
Error: error creating container storage: the container name "doccano" is already in use by "888". You have to remove that container to be able to reuse that name.: that name is already in use
```

在运行`podman-compose up`以后还要`podman-compose down`

```log
Error: unable to start container "b97f90883a23f2583ee2d055317742988f449fd3ba90ff436cf3d096d77b0801": /usr/bin/slirp4netns failed: "open(\"/dev/net/tun\"): No such device\nWARNING: Support for seccomp is experimental\nchild failed(1)\nWARNING: Support for seccomp is experimental\n"
```

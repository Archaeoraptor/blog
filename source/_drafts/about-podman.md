---
title: 容器、已经
date: 2020-11-26 17:32:20
tags:
 - podman
abbrlink: 'about-podman'
---

以及其他的一点容器、云相关的东西

<!-- more -->

## 由Docker想到的

前几年容器火起来的时候，Docker也算是大红大紫的网红公司。但是想挣钱吃独食，谷歌和红帽也不答应。
后来谷歌k8s凭借丰富的经验风头盖过Docker Compose，红帽也不理Docker,自己搞了一套podman，然后几家公司纷纷开始推OCI

这几年

## podman

之前听早就听过podman的安利，一直用着Docker，后来用上了AUR仓库，装东西过于方便连Docker也不怎么用了。后来Docker和透明代理冲突才换的podman。

在自己的Arch上用了几个月，结果发现虽然没了Docker的很多问题（比如root权限啊、乱改iptables啊、），但是还有不少其他的小问题（当然）

### podman使用 VSCode Docker 扩展

### 一些报错和小问题

现在已经不

#### 一些报错

```log
Error: error creating container storage: the container name "doccano" is already in use by "888". You have to remove that container to be able to reuse that name.: that name is already in use
```

在运行`podman-compose up`以后还要`podman-compose down`

```log
Error: unable to start container "b97f90883a23f2583ee2d055317742988f449fd3ba90ff436cf3d096d77b0801": /usr/bin/slirp4netns failed: "open(\"/dev/net/tun\"): No such device\nWARNING: Support for seccomp is experimental\nchild failed(1)\nWARNING: Support for seccomp is experimental\n"
```

再次报错：

```log
Error fetching blob: invalid status code from registry 404 (Not Found)
Error: Error reading blob sha256:aeb7866da422acc7e93dcf7323f38d7646f6269af33bcdb6647f2094fc4b3bf7: Error fetching blob: invalid status code from registry 404 (Not Found)
```

看到一个相关的issue但是没有解决

https://github.com/containers/buildah/issues/2293
https://bugzilla.redhat.com/show_bug.cgi?id=1695236#c14 

据说已经fixed了但是我这里没有

貌似是crun导致的报错

https://github.com/containers/podman/issues/8687


```
ERRO[0000] The storage 'driver' option must be set in /etc/containers/storage.conf, guarantee proper operation. 
ERRO[0000] The storage 'driver' option must be set in /etc/containers/storage.conf, guarantee proper operation. 
Error: error stat'ing file `/home/zhixi/.config/cni/net.d`: No such file or directory: OCI not found
```

把runsc或者runc作为crun的替代就好了


---
title: Nginx部署网站
tags:
  - Nginx
  - Hexo
abbrlink: '1609'
date: 2019-11-05 10:36:05
---
等待填坑
<!-- more -->
部署教研室的小项目的时候接触了一点Nginx，好像Nginx适合部署静态资源和反向代理，决定把博客也试着用Nginx迁到我的VPS上。Netlify在国内的访问速度实在是...

先把Nginx、npm包、git、hexo之类的都装好
（可能还要装gcc和g++用来make）

然后用 Hexo s 试试，其实这时候就启动了。每日浏览量不破千，瞬时浏览量不破百的小博客其实用不着Nginx，用[hexo自带的server](https://hexo.io/docs/server.html)完全应付得过来

这里有一个Nginx的配置网站

<https://nginxconfig.io/>

我无需多言了

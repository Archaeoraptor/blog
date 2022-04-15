---
title: Flask注册登陆有延迟
tags:
  - bug
  - flask
categories:
  - 杂七杂八
abbrlink: 'flask-'
date: 2019-09-19 11:05:44
---
不知道该分到哪一类
<!-- more -->

## Flask注册登录有延迟

之前师兄写的一个数据标注的Flask小网页，拿到之后注册后数据库里有用户名和密码，但是登录就显示未注册，之前注册的是可以的。等了很久之后，用户账号能用了，延迟了几分钟。有的时候等了几个小时也不行，重启也不行，很迷。

发现MySQL和Docker的时间不对，差了8个小时，怀疑是时区的问题
使用`date -R`发现Ubuntu服务器的时间对的，MySQL和Docker时间不对

```mysql
mysql> set global time_zone = '+8:00';
mysql> select now();
mysql> SELECT @@global.time_zone, @@session.time_zone;
```

结果还是MySQL时间不对，但是时区显示是+8:00
重启又试了一下`SET GLOBAL time_zone = 'Asia/Shanghai';`

给Docker同步时间，先用`Docker exec -it <container id>`进入容器，再查看时间，发现Docker时间也差了8小时

修改时间：`docker cp /etc/localtime <container id>:/etc/localtime`

把时区都调成一致之后还是有问题。

这个先等一下，有空再看看师兄的源码，好像还是前后端分离的，有点麻烦。师兄们也不知道是哪的问题，之前都是好的。而且放在教研室服务器一段时间之后也会登录不了，需要重启，不知道是哪的问题。这个坑等有空了再看看。
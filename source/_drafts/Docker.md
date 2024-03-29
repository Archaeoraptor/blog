---
title: Docker笔记
tags:
  - Docker
categories:
  - 笔记
abbrlink: 69d4
date: 2019-10-18 21:31:55
---
开始折腾Docker，等待填坑
<!-- more -->

好耶，是容器。坏耶，是容器。

## 基本设置和操作

### Docker设置

一定要先启动, 不然有可能`Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?`

```bash
service docker start
```

使用`docker exec -it XXXXXX bash`进入容器的命令界面，输入exit退出

同步Docker容器和宿主机的时间：`docker cp /etc/localtime <container id>:/etc/localtime`

加当前用户到用户组里面

```bash
sudo gpasswd -a ${USER} docker
```
#### 离线装Docker镜像

教研室服务器的网一直出问题，还有一台机子没有外网
决定离线搞Docker镜像
发现Docker有离线的方法，就是直接在别的机子上pull，然后

```shell
docker save -o mysql.tar mysql
```

把生成的tar文件拷到服务器上

```shell
docker load -i  mysql.tar
```

### Docker Compose 部署

当部署多个Docker容器，它们之间还有关联的时候，就用Compose代替Docker run，比如师兄Flask写的标注工具，就是前后端分别一个Docker容器

运行：`docker-compose up`自动创建

### Docker镜像

Docker镜像（image）也是一堆文件，可以自己建或者从远程pull别人建好的
有了Docker镜像，再写好Dockerfile就可以跑起来了（跑起来之后就是个容器container）

### Dockerfile

语法近似bash脚本，填写好之后启动会根据文件里面的配置逐步执行

## 常用命令

| Command                                      | 功能                 |
| -------------------------------------------- | -------------------- |
| Docker build                                 | 创建镜像             |
| Docker create                                | 创建镜像（并不启动） |
| Docker exec                                  | 在Docker里面执行命令 |
| Docker images                                | 查看镜像             |
| Docker pull/push                             | 下载/上传镜像        |
| Docker rm/rmi                                | 删除容器/镜像        |
| Docker export/import                         | 导出导入容器         |
| Docker import/load                           | 导出导入镜像         |
| Docker start/stop/restart/pause/unpause/kill | 略                   |

## 踩坑记录

### 一次弱智的踩坑

各位看官散了吧，都是一堆弱智操作搞的

`Failed to enable unit: Unit file /etc/systemd/system/docker.service is masked.`

执行

```bash
systemctl unmask docker.service
systemctl unmask docker.socket
systemctl start docker.service
```

出现 `Job for docker.service failed because the control process exited with error code.
See "systemctl status docker.service" and "journalctl -xe" for details.`

出现`Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?`

出现`systemd[1]: Failed to start LSB: Create lightweight, portable, self-sufficie`

Docker pull出现`Error response from daemon: Get https://registry-1.docker.io/v2/: dial tcp 34.201.196.144:443: connect: connection refused
`

是镜像源或者DNS的问题，参照[这里](https://yeasy.gitbooks.io/docker_practice/install/mirror.html)改了。（阿里的和中科大的教研室的鬼畜教育网根本不稳，最后换了华为的（要注册账号））

出现`pkg_resources.DistributionNotFound: The 'docker-compose==1.8.0' distribution was not found and is required by the application`

想用Docker-compose结果报了一堆python的错

```log
Traceback (most recent call last):
  File "/usr/bin/docker-compose", line 5, in <module>
    from pkg_resources import load_entry_point
  File "/usr/lib/python3/dist-packages/pkg_resources/__init__.py", line 2927, in <module>
    @_call_aside
  File "/usr/lib/python3/dist-packages/pkg_resources/__init__.py", line 2913, in _call_aside
    f(*args, **kwargs)
  File "/usr/lib/python3/dist-packages/pkg_resources/__init__.py", line 2940, in _initialize_master_working_set
    working_set = WorkingSet._build_master()
  File "/usr/lib/python3/dist-packages/pkg_resources/__init__.py", line 635, in _build_master
    ws.require(__requires__)
  File "/usr/lib/python3/dist-packages/pkg_resources/__init__.py", line 943, in require
    needed = self.resolve(parse_requirements(requirements))
  File "/usr/lib/python3/dist-packages/pkg_resources/__init__.py", line 829, in resolve
    raise DistributionNotFound(req, requirers)
pkg_resources.DistributionNotFound: The 'docker-compose==1.8.0' distribution was not found and is required by the application
```

看到网上的解决方案说

```bash
pip3 install backports.ssl_match_hostname --upgrade
```

结果pip3居然又报错了了

```log
pip is configured with locations that require TLS/SSL, however the ssl module in Python is not available.
Collecting backports.ssl_match_hostname
  Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError("Can't connect to HTTPS URL because the SSL module is not available.",)': /simple/backports-ssl-match-hostname/
  Retrying (Retry(total=3, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError("Can't connect to HTTPS URL because the SSL module is not available.",)': /simple/backports-ssl-match-hostname/
  Retrying (Retry(total=2, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError("Can't connect to HTTPS URL because the SSL module is not available.",)': /simple/backports-ssl-match-hostname/
  Retrying (Retry(total=1, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError("Can't connect to HTTPS URL because the SSL module is not available.",)': /simple/backports-ssl-match-hostname/
```

哇，又是SSL证书的问题

参照[stackoverflow的解决方案](https://stackoverflow.com/questions/45954528/pip-is-configured-with-locations-that-require-tls-ssl-however-the-ssl-module-in)

```bash
sudo -s

apt install libssl-dev libncurses5-dev libsqlite3-dev libreadline-dev libtk8.5 libgdm-dev libdb4o-cil-dev libpcap-dev
```

切到`the folder with the Python 3.X library source code`，一般在`/usr/local/bin`底下的python文件夹
运行

```bash
./configure
make
make install
```

依然不行
怀疑是SSL证书的问题，curl使用ssl也报错
但是openssl、libssl-dev都装了的

```bash
apt install ca-certificates
```

证书没过期

又运行了一下openssh居然报错connect:errno=111

查了一下
>This has nothing to do with SSL. Connection refused means that either there is no server or the connection is blocked by firewall. In your case (before your edit) the server is also plain wrong, i.e. ":443" is no valid server name (hostname missing).

难道就是防火墙的问题吗？

查看了一下网络状态

没问题的工作站

```
tcp        0      0 211.83.111.221:40294    211.83.111.224:443      ESTABLISHED 1825/telnet
```

有问题的服务器

```
tcp6       0      0 :::443                  :::*                    LISTEN      101411/apache2
tcp6       0      0 211.83.111.224:443      211.83.111.221:40294    ESTABLISHED 101415/apache2
```


难道学校的ipv6又炸了？

试试`nc -l 443`

emmm，被占用了

不对，我傻逼了，这是对外的端口啊。。。。

等吧

**更新**是网费没了没充钱的问题......
老蔡师兄不是校内的网连不上也是没充钱学校网管给断外网了......
前几天那个Ubuntu Desktop的工作站最近也没钱了，但是http在终端可以用，镜像源什么的也能连，但是那个浏览器一打开就提示我登录账号。
突然意识到了不对，怀疑服务器的网也没钱了。
其实之前一直怀疑是被制裁了，但是工作站就是好的，才一直以为是SSL证书之类的问题
充钱之后，神奇的都能用了
看来是http神奇的逃脱了没钱的制裁
行吧，在这里排查了四五天，期间还重装过好几次Ubuntu Server
果然还是得充钱啊

## 参考

[1]:Docker从入门到实践(https://yeasy.gitbooks.io/docker_practice/)

## 再看Docker

被人骗入坑Docker，甚至教研室服务器上基本全是Docker跑的一个个容器，甚至MySQL都是跑在Docker上的。



<http://dockone.io/article/660>
<https://jimmysong.io/docker-handbook/>

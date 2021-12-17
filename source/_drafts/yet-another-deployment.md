---
title: 又是一个项目上线
date: 2021-11-11 21:25:29
tags:
abbrlink: 'an-unhappy-deployment'
---
不要总是改需求了好不好, 都改了一年了
<!-- more -->

## 74块钱一年的灵车腾讯云

客户那边连钱都不想出，域名也不想买，那就正在搞活动的~~良心云~~腾讯云吧。74块钱4G RAM + 80G SSD 还要什么自行车。

哪家云服务没点后门呢是，首先，我们把腾讯云明面上的后门关了（比如stargate），

（主要是我不太想直接重刷系统，要是我自己的VPS我肯定直接VPS2Arch了）

```bash
set -e

# 终止进程
process=(sap100 secu-tcs-agent sgagent64 barad_agent agent agentPlugInD pvdriver )
for i in ${process[@]}
do
  for A in $(ps aux | grep $i | grep -v grep | awk '{print $2}')
  do
    kill -9 $A
  done
done

systemctl disable tat_agent.service --now
rm -f /etc/systemd/system/tat_agent.service

sh /usr/local/qcloud/stargate/admin/uninstall.sh
sh /usr/local/qcloud/monitor/barad/admin/uninstall.sh
# sh /usr/local/qcloud/YunJing/uninst.sh 这个是老版本的，不需要
rm -r /usr/local/qcloud
rm -rf /usr/local/sa
rm -rf /usr/local/agenttools
```

这样就差不多把腾讯云那一堆多管闲事的后门都删了。

当然就算直接dd也不能阻止云服务厂商宿主机偷数据，还得给数据分区整点LUKS加密之类的东西。不过这小破网站不值钱

### 装MySQL

```
sudo apt install mysql-server
sudo mysql

# 
>create user 'root'@'%' identified by 'ni-di-mi-ma';
>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
>FLUSH PRIVILEGES;
```

然后为了方便某些不愿意用密钥的同学，还是把密码打开（反正这么多漏洞了也不在乎这一个）

```bash
sudo systemctl stop mysql
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
# Uncomment bind-address
```

### 建库

之前教研室上有一个建好的数据库

## Vue打包

### 压缩打包大小

打包出来居然有50M，惊了。这网站不卡还有天理吗。看了一下，占大多数的是图片资源。最大的一个图片有10M左右

## 明文传输密码

好家伙，某位不愿意透露姓名的前端为了省事，直接用POST明文传密码

算了算了，反正又不是我的网站，

## Nginx部署Vue后一片空白

一开始是这样的

```json
    location ~ / {
            root /home/xi102/zhanghaotian/physicalplatform/PhysicalPlatform/qblog-admin/dist/;
            index index.html;
    }
```

部署之后一片空白（Firefox一片空白，Chrome资源加载不全），搜了一下应该是Vue路由的History模式的问题，然后改成这样

```json
location / {
            add_header Cache-Control no-cache;
            root /home/xi102/zhanghaotian/physicalplatform/PhysicalPlatform/qblog-admin/dist;
            index index.html;
            try_files       $uri $uri/ /index.html;
        }
```

然后试了一下Nginx的rewrite，还是不行

```
    location / {
            add_header Cache-Control no-cache;
            root /home/xi102/zhanghaotian/physicalplatform/PhysicalPlatform/qblog-admin/dist;
            index index.html;
            try_files       $uri $uri/ @router;
    }
    location @router {
            rewrite ^.*$ /index.html last;
    }
```

还是不行。于是仔细检查了一下路由的配置，怀疑路由没有问题。

Chrome网页资源加载不全，但是F12的console输出没有任何报错。
又打开了Firefox, 这次发现F12里面有大量的输出信息。找到了关键的一条报错

```
was not loaded because its MIME type, “text/plain”, is not “text/css”.
```

然后搜到了StackOverflow上的回答，加一句`include mime.types;`在`http { ... }`这个块里面

```

```


## Nginx配置Vue跨域问题

## 端口占用

```
Job for nginx.service failed because the control process exited with error code.
See "systemctl status nginx.service" and "journalctl -xe" for details.
```

看了一下日志，端口占用了。

```
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: failed (Result: exit-code) since Sun 2021-10-24 13:27:37 UTC; 10s ago
       Docs: man:nginx(8)
    Process: 1312631 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 1312642 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=1/FAILURE)

Oct 24 13:27:35 xi102 systemd[1]: Starting A high performance web server and a reverse proxy server...

Oct 24 13:27:37 xi102 nginx[1312642]: nginx: [emerg] bind() to 0.0.0.0:9541 failed (98: Address already in use)
Oct 24 13:27:37 xi102 nginx[1312642]: nginx: [emerg] still could not bind()
Oct 24 13:27:37 xi102 systemd[1]: nginx.service: Control process exited, code=exited, status=1/FAILURE
Oct 24 13:27:37 xi102 systemd[1]: nginx.service: Failed with result 'exit-code'.
Oct 24 13:27:37 xi102 systemd[1]: Failed to start A high performance web server and a reverse proxy server.
```

然后看了一下




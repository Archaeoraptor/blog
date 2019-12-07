---
title: Linux从入门到入土
tags: linux
categories:
  - 笔记
abbrlink: '193'
date: 2019-10-11 16:48:51
---
等待填坑
<!--more-->
## 基本的Shell命令

### 系统状态

| Command | Function |
| -----| ----- |
| reboot | 重启 |
| poweroff | 关机 |
| weget  | 下载文件 |
| ps  | 监视进程 |
| top | 动态监视进程 |
| kill | 终止进程 |
| free | 查看内存 |
| history | 查看历史命令 |

### 文件操作

cd, ls, mkdir, ls, mv， rm这些不说了

| Command | Function |
| -----| ----- |
| pwd | 查看当前目录 |
| stat | 查看文件信息 |
| diff | 比较差异 |
| touch | 创建空白文件 |
| tar | 打包文件 |

tar -c 创建压缩， -x 解压缩(-xvzf)
如：

```shell
tar -czvf filemane.tar.gz
```

查看目录下文件大小（包括文件夹）

```bash
du -sh *
```

备份同步rsync, -a（archive）递归并同步时间戳和权限，-v显示详细信息，-z压缩

```bash
rsync -avz source destination
```

### 查找

grep查找文件内的内容，支持正则表达式（但是需要加`-P`参数支持perl正则，否则支持程度有限）

```shell
grep -n include test.c
```

显示行号和查找结果1:#include<stdio.h>

find查找文件

## Vim基础操作

~~不得不用linux命令界面编辑文件又不喜欢vim直接nano就好了~~

Vim有Normal（普通模式），Insert（插入模式）和 Visual（可视模式）

打开是Normal模式，按i进入插入模式，按v进入Visual模式，按ESC返回Normal模式

输入vi test.c创建文件并进入vim，按i进入insert模式，写完保存退出按ESC，:wq保存退出，:q! 不保存退出

文件写入可能需要sudo，不然会E212错误，退出获取管理员权限再保存即可

## 简单的脚本

脚本文件通常是.sh后缀，在执行的时候`bash filename.sh` 。在第一行声明执行的shell解释器路径，如#!/bin/bash 。
接收参数用$, $1,$2,$3为第1、2、3个参数，$?为上一个参数的返回值。
使用逻辑判断&&和||, 输出用户是否管理员：

```shell
[ ! $USER = root ] && echo "user" || echo "root"
```

if 语句, 判断是否存在文件路径，不存在就创建：

```shell
#!/bin/bash
DIR="/media/cdrom"
if [ ! -e $DIR ]
then
mkdir -p $DIR
fi
```

while语句:

```shell
while
do
---
done
```

猜数字

```bash
#!/bin/bash
PRICE=$(expr $RANDOM %1000)
TIMES=0
echo "Guess a number between 0 and 999"
while true
do
        read -p "Input your number" INT
        let TIMES++
        if [ $INT -eq $PRICE ] ; then
                echo "Congratulations, you are right, the number is $PRICE"
                echo"You have guessed $TIMES"
                exit 0
        elif [ $INT -gt $PRICE ] ; then
                echo"Too High"
        else
                echo"Too low"
        fi
done
```

## 不建议轻易尝试的作死行为

下列行为如果你读到这里试了出了麻烦，*本人概不负责*, 在虚拟机和非生产环境里也可以试着玩
卸载自带python:`sudo apt uninstall python`
在wsl中安装KDE桌面
卸载iptables使用`sudo apt remove iptables`

## 各版本体验

### Ubuntu

中规中矩，听说19都出了，没试过，在用18，教研室服务器是16的

### wsl

暂时不支持GPU，指望跑个tensorflow暂时没希望
wsl2已经出了，但我还没升级，到时候再试试2吧

### Manjaro

听说不错，也没有Arch那么恐怖，感觉体验比Ubuntu还好啊

### Debian

VPS上跑的是这个，感觉也还好，用的也不多

## 文件系统

“一切皆文件”, 这是UNIX最初的构想，但是Linux里面并不是，比如socket，真正实现了一切皆文件是贝尔实验室的Plan9,然而已经凉了[^2], **真·入土**
虽然入土了，那个Plan9的风格影响后来很多东西，了也影响了Go，你看LOGO，多像啊
（不得不说贝尔实验室那帮人当年搞出来那一堆东西真的有点意思，形式化验证和Lisp之类的也让人仰望，但是没能商用推广，曲高和寡，~~其实也不是他们的问题，是大多是人都太菜了，比如我~~现在一个个都入土了）

创建文件链接(硬链接和符号链接)

```shell
ln <source> <target>
ln -s <source> <target>
```

加 -s 是符号链接，就像一个指针，源文件删除，符号链接会指向不存在的位置

装载文件（将一个设备添加到文件系统中）
一开始会将/root或者/boot自动装载，装载其他的设备一般用mount命令，比如
`mount /mnt/cdrom`

## 其他

### 支持中文和UTF-8

```shell
    sudo apt-get install language-pack-zh-hans
    sudo vim /etc/environment
```

写入
>LANG="zh_CN.UTF-8"
LANGUAGE="zh_CN:zh:en_US:en"

```shell
sudo vim /var/lib/locales/supported.d/local
```

写入
>en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
zh_CN.GBK GBK
zh_CN GB2312

```shell
sudo locale-gen
sudo apt-get install fonts-droid-fallback ttf-wqy-zenhei ttf-wqy-microhei fonts-arphic-ukai fonts-arphic-uming
```

### 保持SSH连接不自动退出

用的Mobaxterm，直接在settings>SSH>SSH keepalive
当然通常的做法是每隔一定时间向服务器发送心跳[^1]，应该是防止防火墙把SSH干掉，通常SSH不会自己断的
在`/etc/ssh/sshd_config`里面设置ClientAliveInterval和ClientAliveInterval
也可以修改`.ssh/config`文件，设置

```log
Host Server
    ServerAliveInterval 20
    ServerAliveCountMax 9
```

### 设置用户组和权限

root用户UID为0，有生杀大权（我们教研室居然每人都有root用户密码，还日常root权限运行命令，可还行（虽然一共也没几个人用）），也有背锅重任，不想背黑锅是不能乱用的
添加用户（root添加其他用户）

```bash
useradd -u 1998 zjk
```

这样就在/home/zjk创建了一个名为zjk的用户
切换用户的时候使用su命令就行了(Substitute User)
将用户加到root用户组(需要root权限)

```bash
usermod -G root zjk
```

更新，改了之后发现并不好使，要这样

```shell
chmod u+w /etc/sudoers
vi /etc/sudoers
# 加一行
zjk ALL=(ALL) ALL
```

## 参考

[Vim中文社区](https://vim-china.org/)
[Linux工具快速教程](https://linuxtools-rst.readthedocs.io/zh_CN/latest/index.html)
[Linux工具快速教程](https://linuxtools-rst.readthedocs.io/zh_CN/)

## 注

[^1]:[Linux使用ssh超时断开连接的真正原因](http://bluebiu.com/blog/linux-ssh-session-alive.html)
[^2]:[Plan9效应](http://www.di.unipi.it/~nids/docs/the_plan-9_effect.html)

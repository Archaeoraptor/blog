---
title: 你好，Manjaro。再见，Manjaro。
tags:
  - manjaro
  - Archlinux
abbrlink: ff0e
date: 2020-01-17 10:41:09
categories:
  - 不务正业系列
---
洗手.jpg
<!-- more -->

虽然有一些办法不重装从Manjaro跑路到Arch，
<!-- 
比如reddit上这篇帖子[Guide for Manjaro to Arch migration (the dirty way)](https://www.reddit.com/r/ManjaroLinux/comments/jx42ar/guide_for_manjaro_to_arch_migration_the_dirty_way/)
 -->
但是你会遇到各种Manjaro魔改的遗留问题（尤其是加入了相当多魔改的最新版Manjaro,据说很早以前比较容易直接滚成Arch）,**the dirty way**。

我甚至觉得上面这个方法想得到一个纯净的Arch，难度比你去arch的[archive](https://archive.archlinux.org/iso/)下载一个10年前的iso（真正的有图形界面的arch安装镜像哦，官方的）再借助[Arch Linux Archive](https://wiki.archlinux.org/title/Arch_Linux_Archive)（Rollback Machine）等一点一点滚到最新都麻烦。（至少去年Arch群友们玩过这个活动，勉强可行）

直接重装一个原味Arch应该是最快的，也是后遗症最小的。总有人觉得直接不重装换源成Arch比较简单，很遗憾，这其实是最麻烦的一种方法。天上不会掉馅饼。

## 更新--再见Manjaro，

前一阵Manjaro社区出的争吵让我跑路了，Manjaro之前用的还比较舒服（当然直到现在都比较舒服），以后就不知道怎么样了。外面Arch社区骂声一片，里面还总在内斗，不断有人出走，前几周连jonathon都气走了（跑去隔壁endeavour os了）

jonathon的声明在[这篇帖子](https://forum.manjaro.org/t/change-of-treasurer-for-manjaro-community-funds/154888)

过程大概是这样的，philm（Manjaro的CEO，反正Manjaro社区的不少人都觉得此人非常讨厌）挪用社区赞助捐款2000欧元，用于给自己买的电脑。这次购买并没有跟掌钱管家jonathon商量，也没有在社区内征得同意。jonathon觉得按照规定这钱只能用于Manjaro项目的网站建设、项目推广之类的，不是给你们恰烂钱的，然后在论坛发帖子怼philm，然后帖子被删了一次，jonathon的管理员也被撤了。帖子下面众人纷纷表示philm不回应道歉以后再也不捐助Manjaro了，不少人表示以后再也不用Manjaro了，并考虑跑路到KDE Neon、Kubuntu、Arch、Endeavour OS。

philm的回应闪烁其辞，一直说事情不是大家想的那个样子的，但是大家怼他让他公布他和jonathon的完整聊天记录他又不肯。最后philm屁事没有继续担任Manjaro的领袖，jonathon跑路去了Endeavour OS。

## 再次更新--pamac不慎向aur官网打出了DDoS

Manjaro的Pamac不小心DDoS了aur.archlinux.org， 每秒40多来自pamac的请求，aur遭到暴击。
很喜欢reddit评论里的一句话：Just when I thought I couldn’t hate manjaro any more.

https://www.reddit.com/r/archlinux/comments/mz3biz/is_the_aur_down_for_everyone/  
https://gitlab.manjaro.org/applications/pamac/-/issues/1017  

## 关于社区文化和商业化

微软的商业化非常成功，苹果和安卓也是。

即使Linux桌面发行版这种反商业风气比较浓的地方，商业化也不意味着坏事。

REHL也还算不错，名声也比较好；Canonical争议比较大，虽然不招Debian等开源社区的喜欢，早年间也算对推广Linux桌面有贡献，后面snap等有点evil化了

当然，或许还要算上国内的Deepin。（奔着Windows的竞品去的，和其他Linux社区的画风和定位不太一样，倒是和安卓的定位挺像，总不是坏事）

然而Manjaro的商业化我很不看好，就我目前看到的，对上游没有太多贡献，反而给上游带来了更多的麻烦。而且还有一点，Manjaro挪用社区捐款到公司口袋里面，虽然都是Manjaro，但是这在眼里揉不得沙子的很多开源社区人严重这已经是重罪了。当然，也不全是坏事，就不少人的体验来看，至少用起来比Ubuntu舒服不少。

红帽的商业化就很成功，一开始就是给企业提供服务器支持等商业化内容，挣了钱反哺Linux社区。Deepin有不少情怀，比某些充满国家经费和金钱气息的发行版好不少，打包了很多deepin-wine也好DDE也好争取到国内不少公司配适Linux也好，都还算不错。Deepin虽然也带来了大量的小白，但是没有涌到Debian等社区去造成很多麻烦。

我们重温一下“b站或许会倒闭但绝不会变质”的典中典（不知不觉蒙古上单发表重要讲话也一年了，笑）。为什么b站就被骂的这么厉害呢？优酷土豆乐视爱奇艺哪个广告不比这个多，开头的广告还无法跳过。论视频质量和推荐无营养的视频、还有强制使用手机app、以及抄袭和营销号，跟抖音快手比现在的b站已经算良心了吧。  
就像早期的b站那样，这是一群小圈子和爱好者，首先这不是一个商品，这是一个社区，没多少营销号也没有广告，很多人是倾注心血去做视频的，感情很深，氛围也很好。（国内其他视频网站可能也就被封的内涵段子“段友”（现 皮皮虾）有早期b站用户那样类似的归属感了，什么？a站？a站还活着啊？）
当然视频网站的运行成本比较高，靠用爱发电一般做不了很大（就算商业化吃香难看都难免亏损，比如爱奇艺），人多了以后商业化是难免的。

相信大家都能看到Arch社区对Manjaro用户的态度，不管是在archlinuxcn论坛或者irc、电报，还是在国外的reddit社区等，……*&（*%&&￥人*&……（此处省略500字）

不少Arch用户和维护者对Manjaro的恶意大概就像b站老用户这样（你看苹果微软不是比Manjaro邪恶多了，但是他们敌视这些的次数可能比Manjaro要少）

Arch背后没有商业化背景，甚至没有红帽或Canonical这样的公司，然而还是有了比背靠大公司的发行版更多的包（算上野包）和更丰富的wiki(可能没有红帽的wiki好，但是中文翻译多)，这都是Arch的维护者和用户们用爱发电一砖一瓦的心血。它不像Windows和Mac或者红帽的RHEL那样一开始就是一个商品，而且Arch的特点也不太适合做服务器或者商业化。（可能gentoo的商业化用途都比Arch多，还是有一点运维在用gentoo当服务器的）Manjaro的商业化，和原来的社区格格不入。很多地方直接和arch的不少理念和作风背道而驰。
不幸的是Manjaro的社区也有不少伸手党，大量安利Manjaro的视频和帖子都只说好用、好看、好安装，绝口不提社区文化和用户责任。（**而且还在教程里让你添加archlinuxcn的源，同时还说Manjaro更稳定**）

曾经的Arch的ISO有GUI安装界面的（10年前），后来就没了，而官方Wiki上的安装指南也对新手不那么友好了（越来越不友好了）。这里面一个比较重要的原因是过滤小白用户和伸手党，提高用户门槛。这个策略应该还是比较成功的，然而这挡不住Manjaro等发行版的用户。（许多完全有能力安装、使用Archlinux甚至为Archlinux做贡献的用户一开始都被吓到了Manjaro，走了一些弯路，兜兜转转又被洗手到了Arch）
今年愚人节当天Arch出了一个[archinstall](https://archlinux.org/packages/extra/any/archinstall/)的TUI安装方式（还在extra仓库里），当时我一度以为是愚人节玩笑。不过试了一下，还是与其他发行版的GUI安装界面相去甚远。

短期看来不会，长期看来应该也不会。可能是无奈之举吧，暂时也找不到太多比提高安装门槛更好的过滤伸手党的办法了。

这种过滤机制类似很多论坛（比如52pojie等）的邀请制、等级制，早期知乎也是这样的邀请制（甚至还要求实名）。前面提到的b站早期也是要答题的（而且题还很难）。
这些做法确实很有效，很多论坛都靠邀请制挡住了伸手党和发广告的。知乎在开放邀请后内容质量的下滑速度经历过的人应该印象很深，b站也是。我还是个真·小学生的时候赶上了许多中文论坛的最后余辉，现在再想起那些倒闭的、变质的、冷冷清清没人说话的论坛曾经高朋满座的热闹的场面还是不胜唏嘘。Manjaro给Arch带来的影响，总让我想起曾经经历过的社区没落过程。。。


## 链接

[Manjaro Controversies](https://rentry.co/manjaro-controversies)  
[【译】Manjaro 的争议](https://originco.de/Manjaro-Controversies/) 源码菊巨的中文翻译  
[Why You shouldn't use Manjaro.](https://www.reddit.com/r/linux4noobs/comments/fn2iq8/why_you_shouldnt_use_manjaro/)  
[Why not Manjaro?](https://www.reddit.com/r/linuxquestions/comments/brrpqr/why_not_manjaro/)  
[Ubuntu 推出的Snap应用架构有什么深远意义? - farseerfc的回答](https://www.zhihu.com/question/47514122/answer/107412549)  

### 他山之石

Arch的社区属性要远远大于工具属性和商品属性，我们看看那些耳熟能详的社区。这里扔了很多知乎的链接，是因为，我觉得“btw I use arch”和部分arch粉丝在众人眼中的快要溢出优越感，和早期（13年前）中期（15年前）的知乎特别像。很奇怪openSUSE社区和gentoo社区等都没这种arch这种说不出来风气，如果非要找个相似的，可能是部分vim党（emacs党类比gentoo）
还能再诞生一个像早期知乎氛围和内容质量一样的社区吗？我觉得不能（限定简中，长城之内）。还能再诞生一个像arch这样的社区吗？我觉得也不能。有些东西是90年代和2000年初错过就不再有的，未来是商业公司、手机app、短视频直播，自由软件和社区、理想、小众社区诞生的机会不多了。（AOSC等地方能看到火苗和种子，然而每次我看到十年前整个互联网的氛围和现在的样子，还是有点悲观）

[哔哩哔哩注册为什么要去做题？](https://www.zhihu.com/question/334066856)  
[知乎最近涌入了很多在校大学生，这会对知乎产生什么影响？](https://www.zhihu.com/question/20269772/log) 这是一个2012年的问题，今天看起来有点苦涩  
[怎么找到知乎早期的经典回答？](https://www.zhihu.com/question/27202508/answer/35827563) 这是一个14年的问题
[佐藤謙一等人的离开是知乎的转折点吗？](https://www.zhihu.com/question/22682573) [佐藤謙一删掉所有知乎记录离开知乎的原因是什么？](https://www.zhihu.com/question/22675796) [葛巾为什么离开知乎？](https://www.zhihu.com/question/21574455) 同样是几个14年的问题
[什么是「知乎遗风」，现在还有哪些知乎用户拥有这种「遗风」？](https://www.zhihu.com/question/426492597/answer/1542633193) 这是一个2020年的问题

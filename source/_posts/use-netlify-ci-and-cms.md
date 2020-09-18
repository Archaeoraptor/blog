---
title: 尝试Netlify自动部署和CMS
updated: 2019-11-30 20:22:52
tags:
  - netlify
  - ci
categories:
  - 杂七杂八
abbrlink: e9d5
date: 2019-11-30 15:58:43
---
又开始了瞎折腾。话说我这种没人看的静态小破网站，真的有必要持续集成、持续部署吗？

<!-- more -->

## 使用Netlify进行自动化部署

不知道怎么都在搞CI/CD（Continuous Integration/Continuous Deployment），Github也搞了个Github Action。看了一圈感觉对于我这种半个月git commit一次备份博客的人并没有什么必要，而且不管是travis ci还是github action或netlify自己的在线build都很慢，博客东西多一点几分钟就过去了（没有特殊需求就想安安静静写个博客就不要搞这个了，本地hexo g，hexo d推到github pages或者netlify就完事了，速度还快）

之前都是在本地生成，hexo g && hexo d 推到Github pages，然后Netlify关联这个仓库自动拉取(好像是用的钩子吧)。备份的时候再 git commit && git push 本地项目。
现在每次 git commit 备份并 git push，让Netlify从Hexo的项目直接在线 Hexo g ，按部就班授权，改仓库，改deploy命令为 hexo clean && hexo generate
结果直接Deploy failed了
报错:

```log
failed during stage 'preparing repo': Error checking out submodules: fatal: No url found for submodule path 'themes/next' in .gitmodules
```

查了一下发现是Netlify会把每一个包含.git的子目录当成是一个submodule，并且每次都要`git submodule update`一下，但是这个next主题的git仓库又没有在在我的.gitmodules配置里面设置为submodule, 那我们直接把themes/next加到.gitmodules里面就好了\[^1]

在根目录（执行hexo g && hexo d的那个目录）新建`.gitmodules`写入

```log
[submodule "themes/next"]
	path = themes/next
	url = https://github.com/theme-next/hexo-theme-next
```

再执行就行了(先`git commit`,再`git push`,再进Netlify试一下)

看到Deploy log：

```log
4:46:21 PM: Minifying js bundle
4:46:22 PM: Minifying css bundle
4:46:23 PM: Minifying css bundle
4:46:36 PM: Finished processing build request in 56.658478141s
4:46:37 PM: Post processing done
4:46:37 PM: Site is live
```

行了，速度还是可以的，build过程56秒（虽然本地只要几秒，Hugo之类的更快）
对比一下本地

```log
INFO  Files loaded in 3.96 s
···
INFO  180 files generated in 2.01 s
```

这个速度可以接受，据说Netlify最近新增了CI的时间限制（官网说是300 included/month，每个月三百分钟），但是我这种不是重度用户的人根本用不完（实在不行本地hexo s调试好了再提交就完事了，Netlify也要恰饭的嘛，每次CI都要重新安装环境）
带宽也有限制，100GB，我这个没人看的小破博客也根本用不了。

还有一个小问题，改成这样之后发现每一篇文章的更新时间都变成了今天，暂时还没找到判定更新时间的原理是什么，一个办法是手动指定时间戳
在每个markdown文件开头

```markdown
updated: '2019-11-30 20:22:52'
date: '2019-11-30 20:22:52'
```

更新，找到了这个时间戳的问题，看这个issue：[The update time of the article is incorrect](https://github.com/theme-next/hexo-theme-next/issues/893)

大概是Git在checkout的时候，没有正确保留时间戳，试了一下，应该就是push上去的时间，而不是修改后add的时间或者commit的时间

>It uses the push time of the last modification of the file.

根据issue里的解决办法，只要在`hexo clean && hexo generate` 之前加上`git ls-files -z | while read -d '' path; do touch -d \"$(git log -1 --format=\"@%ct\" \"$path\")\" \"$path\"; done`就行了

算了，直接关了吧

## Netlify CMS 使用

（多说一句，Netlify官网打开是真的慢啊）
我已经踩过坑了，没有他们吹的那么好，大骗子，这个也就手机/平板写markdown和发布可以将就一下，真的都不如随身带个U盘里面装着博客工程或者markdown体验好
如果不是有多人协作之类的需求就不要乱试了（除非你真的吃饱了撑的没事干

如果你还是要安装：

```bash
yarn add hexo-netlify-cms
```

然后在Netlify里面设置一下\[^2], 开启git-gateway服务并在Snippet Injection加入一行代码

```html
<script src="https://identity.netlify.com/v1/netlify-identity-widget.js"></script>
```

打开yoursite/admin就可以看到了
这个时候Settings->Identity->Registration是开放注册的，我们改成Invite only
然后在设置里面Settings->Identity->External providers里面进行Github授权或者在在identity页面发邮件邀请（是点进你的team里面identity页面输入邮件地址然后点invite users按钮，不是在设置里的identity）
由于我是白嫖用户，Indetity Level 0，什么Audit log之类的都用不了, 设置到这里基本就结束了。

* 这句话是我在Netlify CMS里面打的，试试，貌似这个在线的markdown编辑器并不太好用,感觉跟CodiMD差了好多啊，也没有夜间模式，也不支持Mathjax之类的

行吧，感觉被坑了啊，一点也不好用啊。~~等有空我在教研室服务器上部署个CodiMD玩~~

（或许可以在本地写好了markdown再粘贴上去，这个markdown在线编辑器实在是难用，而且丑）

唯一能看的就是它会自动同步你的repo，但是你下次在本地编辑的时候还得git pull一下，什么时候仓库不在手边的时候能拿来将就一下。（好像可以拿手机之类的操作啊）

什么，你说多人协作？也不太好用，免费用户invite只能五个账号，鸡肋；收费用户太贵了，并不划算。还不如直接建一个git repository多人协作结合CI发布。至于使用Netlify CMS多人一起维护发布网站内容, 行吧，个人博客基本用不到。也许是给不愿意折腾又想建站还想搞个静态博客的小白用的，但是这样的小白真的有1%吗？

当然netilify cms对于静态网站还是有它的意义的，不然也不会在Github有那么多star，但是对于不需要多人协作的个人博客，就别装了吧（已经装了的和不折腾不舒服斯基人士请随意）

## 参考

[Hexo Netlify CMS](https://github.com/jiangtj/hexo-netlify-cms/blob/master/README-ZH.md)

[^1]:[Git 工具 - 子模块](https://git-scm.com/book/zh/v2/Git-工具-子模块) 可能会有切换分支等其他问题
[^2]:<https://www.dnocm.com/articles/beechnut/hexo-netlify-cms/>

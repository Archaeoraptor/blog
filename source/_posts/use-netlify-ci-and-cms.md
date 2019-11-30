---
title: 尝试Netlify自动部署和CMS
date: 2019-11-30T15:58:43.000Z
updated: '2019-11-30 20:22:52'
tags:
  - netlify
  - ci
categories:
  - 博客
---
又开始了瞎折腾。话说我这种没人看的静态小破网站，真的有必要持续集成、持续部署吗？

<!-- more -->

## 使用Netlify进行自动化部署

不知道怎么都在搞CI/CD（Continuous Integration/Continuous Deployment），Github也搞了个Github Action。看了一圈感觉对于我这种半个月git commit一次备份博客的人并没有什么必要，而且不管是travis ci还是github action或netlify自己的

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
带宽也有限制，100GB，我这个没人看的小破博客也根本用不了

## Netlify CMS 使用

（多说一句，Netlify官网打开是真的慢啊）

安装：

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

行吧，感觉被坑了啊，一点也不好用啊。\~\~改天在教研室服务器上部署个CodiMD玩\~\~

（或许可以在本地写好了markdown再粘贴上去，这个markdown在线编辑器实在是难用，而且丑）

唯一能看的就是它会自动同步你的repo，但是你下次在本地编辑的时候还得git pull一下，什么时候仓库不在手边的时候能拿来将就一下。（好像可以拿手机之类的操作啊）

## 参考

[Hexo Netlify CMS](https://github.com/jiangtj/hexo-netlify-cms/blob/master/README-ZH.md)

\[^1]:[Git 工具 - 子模块](https://git-scm.com/book/zh/v2/Git-工具-子模块) 可能会有切换分支等其他问题
\[^2]:<https://www.dnocm.com/articles/beechnut/hexo-netlify-cms/>

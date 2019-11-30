---
title: 尝试Netlify自动部署和CMS
date: 2019-11-30 15:58:43
tags:
 - netlify
 - ci
categories:
 - 博客
---
又开始了瞎折腾。话说我这种没人看的小破网站，真的有必要持续集成、持续部署吗？
<!-- more -->

## 使用Netlify进行自动化部署

不知道怎么都在搞CI/CD（Continuous Integration/Continuous Deployment），Github也搞了个Github Action

之前都是在本地生成，hexo g && hexo d 推到Github pages，然后Netlify关联这个仓库自动拉取(好像是用的钩子吧)。备份的时候再 git commit && git push 本地项目。
现在每次 git commit 备份并 git push，让Netlify从Hexo的项目直接在线 Hexo g ，按部就班授权，改仓库，改deploy命令为 hexo clean && hexo generate
结果直接Deploy failed了
报错:

```log
failed during stage 'preparing repo': Error checking out submodules: fatal: No url found for submodule path 'themes/next' in .gitmodules
```

查了一下发现是Netlify会把每一个包含.git的子目录当成是一个submodule，并且每次都要`git submodule update`一下，但是这个next主题的git仓库又没有在在我的.gitmodules配置里面设置为submodule, 那我们直接把themes/next加到.gitmodules里面就好了[^1]

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

```
yarn add hexo-netlify-cms
```


## 注

[^1]:[Git 工具 - 子模块](https://git-scm.com/book/zh/v2/Git-工具-子模块) 可能会有切换分支等其他问题
---
title: 尝试Netlify自动部署和CMS
date: 2019-11-30 15:58:43
tags:
 - netlify
 - ci
categories:
 - 博客
---
等待填坑
<!-- more -->
又开始了瞎折腾。

之前都是在本地生成，推到Github pages，然后Netlify关联这个仓库自动拉取。现在想让Netlify从Hexo的项目直接在线 Hexo g ，结果直接Deploy failed了
报错:

```log
failed during stage 'preparing repo': Error checking out submodules: fatal: No url found for submodule path 'themes/next' in .gitmodules
```

查了一下说是Netlify会把每一个包含.git的子目录当成是一个submodule，并且试图`git submodule update`, 我们直接把themes/next加到.gitmodules里面就好了

在根目录（执行hexo g && hexo d的那个目录）新建`.gitmodules`写入

```log
[submodule "themes/next"]
	path = themes/next
	url = https://github.com/theme-next/hexo-theme-next
```

再执行就行了(先`git commit`,再`git push`,再进Netlify试一下)

---
title: Hexo博客部署和Next主题设置
tags:
  - hexo
  - bug
  - Next主题
categories:
 - 博客
mathjax: false
abbrlink: bd8f
date: 2019-06-20 22:23:47
---

next主题从5.X版本迁移到7.X，坑很多，慢慢填
5.X主题已经停止维护了，6.X和7.X版本交给了开源社区维护，后续改进了不少，这也造成了迁移坑多
<!-- more -->

## 升级7.X的大坑

NexT主题的顺畅更新好像很难，官方不推荐，之前乱改的5.X主题升级7.2分支合并出现问题，一堆warning，索性推倒重来直接新搞一个7.X。

升级可以直接按照官网方法

     cd themes/next
     git tag -l
     git checkout tags/v6.0.1

spawn failed 问题，删除.deploy_git文件夹并重新部署。
_config.yml配置请注意对齐问题

添加README.md文档可在这个路径下，这样就会显示在首页
themes\%yourtheme%\source\README.md

为文章设置预览:添加
`<!-- more -->`

插入本地图片推荐hexo-asset-image插件,可以看[这里](http://etrd.org/2017/01/23/hexo%E4%B8%AD%E5%AE%8C%E7%BE%8E%E6%8F%92%E5%85%A5%E6%9C%AC%E5%9C%B0%E5%9B%BE%E7%89%87/)

公式问题：原来采用的hexo-math插件，在hexo-inject插件上有坑，看了看官方推荐，换成[新的插件](https://github.com/theme-next/hexo-theme-next/blob/master/docs/zh-CN/MATH.md)。换完记得卸载hexo-inject插件

设置_config.yml时那个rss如果开启，默认空着就好了，不要填true  

    rss: true

像这样就会报错

本地搜索一直转圈圈，看到issue里面说是插件的问题，卸载后重装，神奇的好了

     npm uninstall hexo-generator-searchdb --save
     npm install hexo-generator-searchdb --save

在Next版本7.20以后（如7.30），hexo-generator-searchdb换成了hexo-generator-search，卸载并安装hexo-generator-search，本地搜索就可以正常使用了，不然会出现只能搜索标题内容的bug

     npm uninstall hexo-generator-searchdb --save
     npm install hexo-generator-search --save

似乎那几个背景特效和进度条都会拖慢网页加载速度，不知道什么时候支持原生动画

报错

     YAMLException: duplicated mapping key at line ...

这说明你的key重复了，比如在yaml里同时两段

     calendar:
          calendar_id: <required> # Your Google account E-Mail
          api_key: <required>
     calendar:
          calendar_id: <required> # Your Google account E-Mail
          api_key: <required>
删掉一个calender或者将二者合并就可以了

next v7.3.0 版本的定制路径改了，比如custom.styl改成了source/_data/styles.styl，参见

    custom_file_path:
      #head: source/_data/head.swig
      #header: source/_data/header.swig
      #sidebar: source/_data/sidebar.swig
      #postMeta: source/_data/post-meta.swig
      #postBodyEnd: source/_data/post-body-end.swig
      #footer: source/_data/footer.swig
      #bodyEnd: source/_data/body-end.swig
      #variable: source/_data/variables.styl
      #mixin: source/_data/mixins.styl
      #style: source/_data/styles.styl

你也可以修改custom_file_path:来自定义定制内容路径。
这些内容在git merge的时候需要手动移动

Next v7.3.0 版本的标签不再支持 Full-image tag

     WARN  Full-image tag is no longer supported.

使用

     {% fi https://image-url, 图片名称, , 35% %}

这样的标签无法再进行图片缩放,可以改用html方式

     <img src="图片地址" width="50%" height="50%"  style="margin: 0 auto;"/>
     <img width=400 src=" your-image-link" alt="上海鲜花港 - 郁金香" />
或

    <div style="width: 200px; margin: auto">![picture name](example.jpeg)</div>

或者hexo标签

     {% img [class names] /path/to/image [width] [height] "title text 'alt text'" %}

另外，fancybox虽然现在还支持，但是据说计划在8.0版移除，所有jQuery有关的东西都在逐步移除，目前已经支持 medium-zoom，可以在设置中开启

     npm install medium-zoom

"在新版中 quicklink 不建议再使用，最起码不要与 pjax 同时开" 更新,这个在7.4版本以后修复了
无法加载 woff 字体资源 404 问题

     Failed to load resource: the server responded with a status of 404 () fontawesome-webfont.woff2:1

这个找到提示的文件夹 source/font 路径,找到这三个缺失的文件,补上就可以了(可能没有这个文件夹,新建一下就行了)

## 更新主题

这里我使用的Next主题，参考<https://github.com/theme-next/hexo-theme-next/blob/master/docs/zh-CN/DATA-FILES.md>
在7.X以后大的更新还是需要手动处理冲突，但是Next主题将两个_config.yml文件合并搞了一个next.yml，在source/_data底下。这样更新的时候git pull后commit和merge就不用每次手动处理主题的_config.yml冲突了
merge之后指定版本

```shell
git checkout tags/v7.4.1
```

(可能会有nothing added to commit but untracked files present的提示，运行git clean -f清理不想要的文件，或者加到.gitignore里面, 还要就使用git add . )
如果版本指定不对，会有detached HEAD出现，重新指定正确版本。如果还不行，可能git add等出了冲突，看[这里](https://www.jianshu.com/p/ae4857d2f868)

## 升级Hexo4.0

今天检查更新的时候看到这样的提示，发现Hexo出4.0了。
跑到群里一看，说是什么Hexo 4.0.0 会有巨大的性能提升（相比 Hexo 3.2 生成速度提升 10%，内存占用减少 40%）。闲着没事就升级玩玩吧虽然我这也没几篇文章，生成速度本来也不慢。

```shell
λ npm-upgrade
Checking for outdated dependencies for "D:\zjk\myblog2\package.json"...
[====================] 21/21 100%                                                                                            s can be used.

New versions of active modules available:

  hexo                      ^3.9.0   →   ^4.0.0
  hexo-deployer-git         ^1.0.0   →   ^2.0.0
  hexo-generator-archive    ^0.1.5   →   ^1.0.0
  hexo-generator-category   ^0.1.3   →   ^1.0.0
  hexo-generator-feed       ^1.2.2   →   ^2.0.0
  hexo-generator-index      ^0.2.1   →   ^1.0.0
  hexo-generator-tag        ^0.2.0   →   ^1.0.0
  hexo-renderer-ejs         ^0.3.1   →   ^1.0.0
  hexo-renderer-stylus      ^0.3.3   →   ^1.1.0
  hexo-server               ^0.3.3   →   ^1.0.0

? Update "hexo" in package.json from ^3.9.0 to ^4.0.0? (Use arrow keys)
> Yes
  No
  Show changelog
  Ignore
  Finish update process
```

直接升级

```shell
npm install -g hexo@4.0.0
```

更新，现在hexo-cli包也上hexo4.0了，使用官网的那个带cli的命令就行了
升级之后试一下'hexo generate'

```
INFO  Files loaded in 1.94 s
INFO  140 files generated in 3.07 s
```

好像是快了点
等等，更新了Hexo4.0以后下面的分页突然变成这个样子了，我先报个修
<img src="https://raw.githubusercontent.com/Archaeoraptor/image_resources/ImageofBlog/20191016094709.png" alt="bug" style="zoom:80%;" />

<img src="https://raw.githubusercontent.com/Archaeoraptor/image_resources/ImageofBlog/20191016095007.png" alt="bug" style="zoom:80%;" />

找到了问题了，已经在[issue#3729](https://github.com/hexojs/hexo/issues/3729)里解决了
现在直接升级到master分支就行了

```git
cd themes/next
git pull https://github.com/theme-next/hexo-theme-next master
git checkout master
```

看到显示这样就行了

>Already on 'master'
Your branch is up to date with 'origin/master'.

<img src="https://raw.githubusercontent.com/Archaeoraptor/image_resources/ImageofBlog/foot.png" alt="Picture" style="zoom:80%;" />
好了，开心。目前没看到其他bug了，请放心更（cai）新（keng）

更新，现在最新的master报这样的错：

     ERROR Cannot read property 'replace' of null

还有这样的错

     ERROR Process failed: layout/tag.swig

换到nunjucks分支，也有这个问题，先切到contrib分支里面，nunjucks那块正在施工，大家先用着旧版，我已经报修了。劳模mimi同学已经在修了（19.10.18）
更新，感谢劳模Mimi同学，已经修了[Migrate to Nunjucks [3] #1226](https://github.com/theme-next/hexo-theme-next/pull/1226)

## 绑定域名并托管到netlify

先到namecheap或者namesilo等网站找个中意的域名，不介意在天朝备案腾讯和阿里云买也可以，不愿意花钱也可以申请个免费的。我这里0.99刀一年优惠买了个[zhangjk98.xyz](https://zhangjk98.xyz/)

在netlify的domian setting里找到Custom domains按照提示来，可以参考这个[使用 Netlify 托管个人网站 详细步骤](https://medium.com/@Pudge1996/2018-01-02-7013eddc413c)

## Google Search Console 验证和配置

这里以netlify为例，namesilo等域名商也可进行验证
<img src="https://raw.githubusercontent.com/Archaeoraptor/image_resources/ImageofBlog/20190830141615.png" alt="dns验证" style="zoom:80%;" />
有自己的域名可以直接选择domian验证，看[这里](https://www.woguide.com/archives/4861.html)
打开netlify，找到dns setting，点击下方add new record 按钮
![添加验证](https://raw.githubusercontent.com/Archaeoraptor/image_resources/ImageofBlog/20190830142257.png)
value就填谷歌给你的那个值，等几分钟就好了

## 美化

设置footer跳动的红心
在next配置里找到icon 设成下面这个样子

       icon: 
          # Icon name in Font Awesome. See: https://fontawesome.com/v4.7.0/icons/
          # `heart` is recommended with animation in red (#ff0000).
          name: heartbeat
          # If you want to animate the icon, set it to true.
          animated: true
          # Change the color of icon, using Hex Code.
          color: "#FF0000"

自定义字体: _config.yml里找到

     font:
          enable: true

## 使用caniuse测试浏览器兼容性

{% caniuse fetch %}
{% caniuse sharedarraybuffer @ current %}
{% caniuse link-rel-modulepreload @ past_1,past_2,past_3,past_4,past_5 %}


## 使用yarn管理npm包和travis-ci自动化部署

可选，本地生成速度够快或者不嫌麻烦也没太有必要上travis-ci
不愿意折腾就直接本地生成部署就好了，ci反而慢，而且就想发个博客的话完全没有必要！

今天经人提醒，hexo s 是动态编译的，本地预览直接hexo s 就行了，不用先 hexo g 生成

## 插件选择

### 踩坑

lazyload插件、fancybox插件(改用mediumzoom)、pjax插件、fliter-optimizer插件（经常跟很多地方冲突）等建议慎用

### 给Mardown添加论文脚注效果

安装插件：

```shell
npm install hexo-reference --save
```

注释测试[^1]

## 可能的问题请参考

[Hexo常见问题解决方案](https://shuyelife.github.io/post/Hexo%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88/)
[hexo主题美化](https://dp2px.com/2019/04/25/hexo-meihua/)
[hexo-next主题下的美化](https://blog.yleao.com/2018/0901/hexo-next主题下的美化.html)
[hexo官网](https://hexo.io/)
[next文档(6.0以后)](https://theme-next.org/docs/)
[hexo指南](https://hexo-guide.readthedocs.io) 从node.js操作到next主题，很详细，自己折腾可以参考,也记录了很多坑，但是不全，非官方指南，都是个人博客的坑。他还写了别的[指南](https://www.zhujian.tech/posts/d22ab967.html)
<https://github.com/theme-next/awesome-next#plugins>Next主题的插件，7.X版本之后越来越多的功能都剥离出来做成了插件, 使用时要npm install 一下，不能会直接在设置里设为true
[markdowm教程](https://github.com/LearnShare/Learning-Markdown/tree/v2)

## 注释

[^1]: 这是一个注释测试
---
title: Archlinux下VSCode+Latex+Zotero的论文写作方案
date: 2021-09-24 18:44:24
abbrlink: 'paper-with-latex-zotero-vscode'
tags:
- Zotero
- LaTeX
---
开源拖拉机将就着用吧，又不是不能用，再见Word、EndNote和Windows。顺便说一下你电的LaTeX模板怎么用。不定期更新到毕业。
<!-- more -->

又遇到了Lamport老爷子的东西，上一次看到Lamport~~还是上次~~还是看Paxos算法的时候。

Office很强，但是微软不给Linux开发桌面版（好吧，我有在线的教育版，在线版写写文档还行，但是排版有一些捉襟见肘）。好吧，虚拟机或者wine是可以的但是我不想用。wps的Linux版倒是很不错，但是调格式放到office上又乱了，对数学公式的支持也不是特别好。 
另一个让我放弃word和wps的重要原因是我想要用git做版本控制，手动命名一个个v0.0.8、v0.2.1版本的论文初稿实在是太蠢了。  
最近放弃WPS的原因是新版本实在太糊了，我在Archlinux上试图降级旧版又失败了。本来以为新版WPS字体模糊是因为没有配适4k分辨率的屏幕，这个文章[WPS For Linux个人版更新（Qt5版本）](https://zhuanlan.zhihu.com/p/395993576)的评论去也确实是这么说的。结果我打开1080p的笔记本，发现字体更糊了。  
overleaf应该是我用过最好用的Latex在线编辑，但是我对在线的东西不是很信得过。TexStudio又不是太好看，高分屏下还比较糊，键位也不习惯，放弃了。lyx是很好用，但是我不太喜欢所见即所得，还是习惯左边源码右边编辑结果pdf这种。  

最终又回到了VSCode，目前的方案是ArchLinux下Zotero+textlive+VSCode（LaTeX Workshop插件）

## 安装texlive

我用对中文支持比较好的xelatex，在Arch下我们装texlive的包就可以了。pdflatex也包含在里面，但是这个对中文支持没有那么好，一般用xelatex生成中文pdf。

```bash
pacman -S texlive-core texlive-langchinese  texlive-fontsextra
```

texlive的不少宏包在Arch上都有系统的包，你可以用`yay`等AUR helper装，如果想用`tlmgr`装，那就用[tllocalmgr-git](https://aur.archlinux.org/packages/tllocalmgr-git/)这个AUR包。

## Zotero配置

### 参考文献抓取和管理

首先是中文论文网站引用和pdf抓取的问题，可以用下面这个插件：[Zotero translators 中文维护小组](https://github.com/l0o0/translators_CN)

知网文献可以用这个插件：[jasminum](https://github.com/l0o0/jasminum)

下载并放到translater文件夹中(Linux下默认应该是`~/Zotero/translators`)

```bash
git clone https://github.com/l0o0/translators_CN.git 
cp -r translators_CN/translators/* ~/Zotero/translators
```

参照README更新插件的translater就行了。

VSCode有一个Zotero LaTeX插件，可以用快捷键直接插入文献。安装后报错`Error adding citation: HTTPError: Response code 404 (Not Found)`，见[Zotero Citations: could not connect to](https://github.com/mblode/vscode-zotero/issues/2)。安装beta版本后成功解决。`yay -S zotero-beta`

使用这个插件需要安装zotero插件[zotero-better-bibtex](https://github.com/retorquere/zotero-better-bibtex)

具体使用请参考：[VScode使用说明(Zotero插件) 刘再华](http://xugee.com/images/3/3a/VScode使用说明.pdf)

安装好之后按`Alt+z`就可以方便的插入参考文献了。

然后要插入GB/T 7714格式化的参考文献，可以参考[符合 GB/T 7714-2015 标准的 biblatex 参考文献样式](https://ctan.math.illinois.edu/macros/latex/contrib/biblatex-contrib/biblatex-gb7714-2015/biblatex-gb7714-2015.pdf)

有模板可以直接套模板，模板应该会处理参考文献格式。没有特殊喜好可以用 等支持GB/T格式的模板。

ps：如果单纯为了插入GB/T格式的参考文献可以用一点脏办法: [电子科技大学LaTeX模板参考文献问题解决](https://zhuanlan.zhihu.com/p/141805704) (不得已的办法)

### 自建同步

Zotero的文件同步免费的只有300M，而且很慢。

我们可以自建文件同步，只要支持WebDAV协议就行了。这个好办，Nginx就能做到。

## VSCode配置

### LaTex插件选择

通常推荐装Latex Workshop这个插件。配置可以参考[官方文档](https://github.com/James-Yu/LaTeX-Workshop/wiki), wiki写的很全。功能当然没有TeXStudio全，但是已经满足我的日常需求了（一般只装这一个插件就够了）

另一个插件叫LaTeX。LaTeX Workshop功能比LaTeX插件多一点，如果只想把VSCode当一个普通的文本编辑器不想要额外的snippet等功能的话，装LaTeX这个插件就可以了。

#### 编译配置

编译配置推荐使用`latexmk`的方案，可以省去xelatex-bibtex-xelatex*2的多次编译。  

比如你电的毕业论文模板，完整的编译需要

```bash
xelatex main.tex
bibtex main.aux
bibtex accomplish.aux
xelatex main.tex
xelatex main.tex
```

编译的时候需要先latexmk编译一遍，再bibtex编译一遍，再latexmk编译一遍，见[](https://liam.page/2016/01/23/using-bibtex-to-generate-reference/)

或者参考这篇文章：[我们该如何优雅地使用LaTeX in 2020](https://zhuanlan.zhihu.com/p/329938007)

编辑`settings.json`, 加上能够编译参考文献的配置，就可以在左侧边栏点击运行了。（其实我还是喜欢在命令行里`alias`将这三个命令指定一个短命令）

```json
"latex-workshop.latex.recipes": [
    {
        "name": "xelatex -> bibtex -> xelatex*2",
        "tools": [
            "xelatex",
            "bibtex",
            "xelatex",
            "xelatex"
        ]
    },
]
```

**这样需要编译四次，有一个更好的方法是`latexmk`增量编译**

我们将`settings.json`**改成这个样子**（方案来自[在 VSCode 的 LaTeXworkshop 插件中使用 LaTeXmk](https://liam.page/2020/04/24/using-LaTeXmk-with-LaTeXworkshop-with-VSCode/)）

```json
    "latex-workshop.latex.tools": [
        {
            "name": "LaTeXmk",
            "command": "latexmk",
            "args": [
                "-xelatex",
                "-synctex=1",
                "-shell-escape",
                "-interaction=nonstopmode",
                "-file-line-error",
                "%DOC%",
                "-outdir=%OUTDIR%",
            ]
        },
    ],
    "latex-workshop.latex.recipes": [
        {
            "name": "LaTeXmk",
            "tools": [
                "XeLaTeXmk",
            ]
        },
    ],
```

这样只需要`latexmk`一个编译命令，`latexmk`的增量编译也比原来快了。

### 其他功能

LaTeX Workshop 左侧边栏可以查看字数统计  
可以自定义snippest快捷命令  
可以插入参考文献（需要手打`\cite{}，支持搜索bib文件里的文献（可以用zotero导出并开启同步））, 这样就不用装zotero latex插件了。

### json配置

这是我的`settings.json`配置：

```json
    //双向搜索
    "latex-workshop.view.pdf.viewer": "tab",
    //将编译生成的文件放到build文件夹
    "latex-workshop.latex.outDir": "build",
    //使用latexmk解决插入文献bibtex需要多次编译的问题
    "latex-workshop.latex.tools": [
        {
            "name": "LaTeXmk",
            "command": "latexmk",
            "args": [
                "-xelatex",
                "-synctex=1",
                "-shell-escape",
                "-interaction=nonstopmode",
                "-file-line-error",
                "%DOC%",
                "-outdir=%OUTDIR%",
            ]
        },
    ],
    "latex-workshop.latex.recipes": [
        {
            "name": "LaTeXmk",
            "tools": [
                "XeLaTeXmk",
            ]
        },
    ],
    //去掉右下角烦人的警告
    "latex-workshop.message.error.show": false,
    "latex-workshop.message.warning.show": false,
    "latex-workshop.latex.clean.fileTypes": [
        //设定清理文件的类型(ctrl+alt+c：清除辅助文件)
        "*.aux",
        "*.bbl",
        "*.blg",
        "*.idx",
        "*.ind",
        "*.lof",
        "*.lot",
        "*.out",
        "*.toc",
        "*.acn",
        "*.acr",
        "*.alg",
        "*.glg",
        "*.glo",
        "*.gls",
        "*.ist",
        "*.fls",
        "*.log",
        "*.fdb_latexmk",
        "*.nav",
        "*.snm",
        "*.bcf",
        "*.run.xml"
    ],
    "extensions.ignoreRecommendations": true,
    "zotero.latexCommand": "cite",
    "[latex]": {
        "editor.formatOnPaste": false,
        "editor.suggestSelection": "recentlyUsedByPrefix"
    },
```

## 电子科技大学论文模板

Github上有一个不错的模板是[shifujun/UESTCthesis](https://github.com/shifujun/UESTCthesis)，但是时老师的这个模板很久没更新了。
另一个是还在一直更新的模板，本硕博都有，直接clone下来改一下就好了（你电图书馆推荐的，勉强算是半官方的吧）

```bash
git clone https://github.com/x-magus/ThesisUESTC
```

网上有一些教程，包括：
知乎上的一篇文章[UESTC 本科Latex毕设论文模板 无痛上手指南](https://zhuanlan.zhihu.com/p/126712982)，这个我本科写毕设的时候就看到同学在朋友圈转过。这个教程我不是很推荐，因为：
>但是这里为了减少小伙伴们的学习成本达到“快速无痛上手”的目的（同时考虑到有些小伙伴用word写的很熟练），这里引用电子科大图书馆嵇灵老师的方法：用mathtype编辑，然后转化为latex代码。

河畔现任站长的介绍帖子：[LaTeX 学校官方模板-2019年11月发](https://bbs.uestc.edu.cn/forum.php?mod=viewthread&tid=1786231&extra=&page=1) （需要登陆查看）  
以及上一个帖子附件中嵇灵老师的PPT： 使用Latex模板撰写毕业论文（2019)  
ps：个人不推荐这个PPT，里面推荐使用被思杰克马丁代理的Mathtype，推荐直接使用Office 2019的`Alt+=`手打公式（比LaTeX的公式甚至还舒服一点，也直接支持LaTeX的公式），实在想用图形界面输入公式就用AxMath吧，正版30多，还算良心不。  

上面那些教程的安装和使用几乎都是Windows环境，编辑器是overleaf和texstudio，而且操作偏向图形化界面点点点（Linux用户应该不会喜欢连LaTeX打公式都要点点点再粘贴上去）

注意要安装下面这些额外的包，不然会报`! LaTeX Error: File 'multirow.sty' not found`等错误。

```bash
sudo pacman -S texlive-latexextra texlive-science texlive-bibtexextra
```

### Archlinux下字体问题

一种选择是装win10字体，然后修改`thesis-uestc.cls`的Default字体设置。AUR有个包`ttf-ms-win10-auto-zh_cn`，但是这个包由于再分发的版权原因要下载整个win10的iso，然后解压只保留ttf字体文件。身边有win10的建议直接将ttf文件拷贝过来（只要`simhei.ttf`，`simsong.ttf`，`simkai.ttf`就可以了）。
或者安装方正字体（默认的），AUR有个包`ttf-fangzheng`由于版权原因，也不能直接yay安装，只提供PKGBUILD，需要到官网下载ttf然后手动`makepkg`

另一种选择是找到模板的字体设置，直接修改`thesis-uestc.cls`文件（反正毕业论文只要求宋体和黑体，又没说什么宋体和黑体，用系统默认的黑体和宋体就行了）

```latex
\else
\iflinux
  \setCJKmainfont[BoldFont=FandolSong-Bold.otf,ItalicFont=FandolKai-Regular.otf]{FandolSong-Regular.otf}
  \newCJKfontfamily{\heiti}{FandolHei-Regular.otf}
  \newfontfamily{\heiti@letter}{FandolHei-Regular.otf}
  \setallmainfonts{Times New Roman}
```

除了中文字体还需要一个Times New Roman罗马字体，这个AUR的包可以直接装。

```bash
yay -S ttf-times-new-roman
```

### 编译

直接运行`latexmk main.tex`就可以了。

在VSCode里面装了LaTeX Workshop可以点击左上角那个绿色的运行按钮。会弹出一个pdf，将pdf放到右边一栏

### 关于你电的一些问题

什么？教务处非要收word怎么办？要不妥协用Word，要不用pdf转图片转Word随便交一个上去，要不就pandoc顶着格式乱掉硬转一个，反正这个就是规定留档，没人看。  
（隔壁西电早都支持提交LaTeX论文呢，不会有的985连这都不支持吧，非要冒着被制裁的风险用微软家的Word？不会吧？）

老板要用Word批注功能怎么办？这个没办法，回去用Word吧。
推荐你畔现任站长的一篇文章：[适用于毕业论文的word排版技巧](https://github.com/CuiaCuiSha/UseWordInThesis) 之前本科毕设论文排版就看的这个，比你电图书馆的讲座教程和PPT好多了。

知网查重LaTeX会查公式，查重率高怎么办？不好意思这个没办法。

## LaTeX教程和模板推荐

### 模板

LaTeX本来就是用于排版的专业排版工具，能做出很多相当漂亮和复杂的效果。LaTeX能做到什么程度可以看StackExchange上的这个问题：[Showcase of beautiful typography done in TeX & friends](https://tex.stackexchange.com/questions/1319/showcase-%20%20of-beautiful-typography-done-in-tex-friends)

想要自己从0设计排版成这样的效果是需要肝的。而我用LaTeX是因为不想花太多精力在排版上，想把它当成Markdown一样的东西用。所以我需要模板。

模板可以来这里找：[LaTeXStudio](https://www.latexstudio.net) 不过这个网站最近突然换了前端页面还多了不少广告（有种不详的预感）  

### 教程

教程这个不同人适合不同的教程，下面是我比较喜欢的一个：

[一份其实很短的 LaTeX 入门文档](https://liam.page/2014/09/08/latex-introduction/)

## 其他问题

### 关闭LaTeX Workshop右下角的烦人通知

LaTeX WorkShop插件每次保存都会弹出通知`Formatting failed. Please refer to LaTeX Workshop Output for details.`

关闭右下角每次都弹出的烦人提示

```json
    "latex-workshop.message.error.show": false,
    "latex-workshop.message.warning.show": false,
```

### 双向跳转

~~双向奔赴了属于是~~

目前最新的LaTeX Workshop插件是无需配置可以直接跳转的。（插件内置的PDF.js，在VSCode内预览）  
网上很多教程已经过时了，直接看官方文档。官方文档写的很清楚：<https://github.com/James-Yu/LaTeX-Workshop/wiki/View#synctex>

从pdf跳转到tex文件使用`Ctrl+鼠标左键`  
从tex文件定位到pdf相应位置`Ctrl+Alt+J`  

### 插入图片

插入图片可以放到相应位置然后手写`\includegraphics{example.jpg}`。  
如果想从剪切板自动粘贴，和Markdown一样，用paste image插件就行了。我Markdown的paste image配置和latex的不一样，所以放到workspace的json文件里

```json
{
    "pasteImage.defaultName": "X",
    "pasteImage.encodePath": "none",
    "pasteImage.path": "${currentFileDir}/pic",
    "pasteImage.showFilePathConfirmInputBox": true,
    "pasteImage.insertPattern": "\\includegraphics[scale =]{${imageSyntaxPrefix}${imageFilePath}${imageSyntaxSuffix}}"
}
```

或者我们在user的`settings.json`里加上

```json
"[latex]": {
    "pasteImage.insertPattern": .......
}
```

表示只对latex生效。

insertPattern选项请根据模板自己修改，比如你电的学位论文模板改成这样就可以了。

```json
{
    "pasteImage.insertPattern": "\\begin{figure}[h]\n\t\\includegraphics[width= 6cm]{${imageFileName}}\n\t\\caption{${imageFileNameWithoutExt}}\n\t\\label{fig:${imageFileNameWithoutExt}}\n\\end{figure}",
}
```

另外需要注意的是paste image插件的快捷键`ctrl+alt+v`和LaTeX Workshop冲突，要在`Keyboard Shortcuts`里面换掉一个，不然快捷键粘贴图片不起作用。

### 参考文献报错

设置参考文献这两行要放在`\end{document}`前面

```latex
\bibliography{export.bib}
\bibliographystyle{plain}
```

不然会报错

```log
I found no \bibdata command---while reading file test.aux
I found no \bibstyle command---while reading file test.aux
```

### 某些知网文献bibtex编译报错

垃圾知网不仅弄出了天价查重、caj格式这些天怒人怨的东西，中文文献的引用格式也挺乱的，经常缺这个少那个，而且不支持bibtex。如果用zotero转出来也会因为缺少各种字段而报错。查了一下没什么太好的解决办法。
[如何使用BibTeX引用中文参考文献？ - 刘海洋的回答 - 知乎](https://www.zhihu.com/question/26398909/answer/32643788) 刘海洋老师推荐自己写bibtex，但是我给这些东西写这些，我用LaTeX就是因为不想在word的排版上纠缠过多。  

百度好歹干了点人事，能导出bibtex格式。但是有很多学位论文，百度学术也没有万方也没有，就知网有。  

之前说过，如果单纯为了插入GB/T格式的参考文献可以用一点脏办法: [电子科技大学LaTeX模板参考文献问题解决](https://zhuanlan.zhihu.com/p/141805704) (不得已的办法)。比如插入知网文献的时候可以用这个。**一开始觉得这是不太好的脏办法**，在被知网的一些参考文献格式整麻了以后，觉得**这才是最好的方法**。直接粘贴纯文本，反正最后排版效果一样就行了。bibtex的理念不适合处理脏东西，脏东西要用脏办法。

编辑`thesis-uestc.bst`, 新建一个参看文献格式

```bst
FUNCTION {biaoti}
{
    bibitem.begin
    title write$
    newline$
}
```

然后在`reference.bib`里面粘贴复制粘贴的文本就可以了，相当于引用纯文本

### 自动换行

只对Latex开启自动换行

```json
"[latex]": {
    "editor.wordWrap": "on"
}
```

#### 电子科技大学论文模板识别不了linux环境

编译的时候加参数`-shell-escape`。

## 链接

<https://github.com/zuicy/UESTC_report_latex/tree/master/UESTC_report> 电子科技大学实验报告的LaTeX模板，是你电信软学院的同学做的，看起来似乎全校通用（改一下学院名字就好了）。  
LaTeXStudio上那个是比较老的模板：[电子科大的实验报告 LaTeX 模板 - 用户投稿](https://www.latexstudio.net/archives/51541.html)

[Linux 下的 LaTex 写作工具链（1）](https://zhuanlan.zhihu.com/p/111889252)

[VScode使用说明(Zotero插件) 刘再华](http://xugee.com/images/3/3a/VScode使用说明.pdf)  

<https://github.com/mblode/vscode-zotero> vscode zotero的Markdown引用文献插件。  

[使用 BibTeX 生成参考文献列表](https://liam.page/2016/01/23/using-bibtex-to-generate-reference/)

<https://en.wikibooks.org/wiki/LaTeX>

<https://github.com/hushidong/biblatex-zh-cn>

<https://github.com/hushidong/biblatex-solution-to-latex-bibliography>

<https://liam.page/2020/04/24/using-LaTeXmk-with-LaTeXworkshop-with-VSCode/>

<http://ddswhu.me/posts/2018-04/vs-code-for-latex/>

<https://www.tug.org/texlive/doc/texlive-zh-cn/texlive-zh-cn.pdf>

---
title: Typst真好用啊
date: 2023-12-2 17:58:27
tags:
- typst
categories:
- 
---
LaTeX is too sophisticated，markdown is too simple
<!-- more -->

Typst作为苦lAtEx久矣的产物，使用体验比LaTeX简单不少，该有的排版功能也基本都有了。typst现在的更新很活跃，issue区回复也很快

（当然写博客还是用markdown，这玩意作为html青春版，和前端以及浏览器的相性太好了）

Typst一开始是网页版产品，后来才开源的。网页版的体验比vscode等本地编辑器装插件要好不少。但是，当然是选择本地安装啦。我对于网页版的东西天生信不过，当然是存到本地的硬盘里才放心。

```bash
yay -S typst typst-lsp
```
然后VSCode可以装Typst LSP和Typst-Preview插件，然后直接用就好了。可以在侧栏打开PDF Preview，像LaTeX和markdown插件那样。

template.typ

```typst
// The project function defines how your document looks.
// It takes your content and some metadata and formats it.
// Go ahead and customize it to your liking!
#let project(title: "", authors: (), logo: none, body) = {
  // Set the document's basic properties.
  set document(author: authors, title: title)
  set page(numbering: "1", number-align: center)
  set text(font: "Linux Libertine", lang: "en")

  // Title page.
  // The page can contain a logo if you pass one with `logo: "logo.png"`.
  v(0.6fr)
  if logo != none {
    align(right, image(logo, width: 26%))
  }
  v(9.6fr)

  text(2em, weight: 700, title)

  // Author information.
  pad(
    top: 0.7em,
    right: 20%,
    grid(
      columns: (1fr,) * calc.min(3, authors.len()),
      gutter: 1em,
      ..authors.map(author => align(start, strong(author))),
    ),
  )

  v(2.4fr)
  pagebreak()


  // Table of contents.
  outline(depth: 3, indent: true)
  pagebreak()


  // Main body.
  set par(justify: true)

  body
}
```

或者手动compile：

```bash
typst compile demo.typ
```

## 字体和中文排版问题

目前typst对非latin字符的支持有限，处于勉强能用的样子

```bash
# 显示可用字体
typst fonts
```

## 链接

<https://typst.app/>  
<https://github.com/typst/typst>  
---
title: 前端基础知识学习
tags: 笔记
categories:
  - 笔记
abbrlink: 4a8d
date: 2019-11-18 15:58:43
---
开始补习
<!--more-->
写markdown偶尔要用html的标签，折腾这个博客一堆前端知识，想写个爬虫玩也要在浏览器抡起F12，教研室那个项目也要折腾js，这样下去不行，得恶补前端了。
先从html，css，javascript三个补起吧

```text
   1. HTML to define the content of web pages

   2. CSS to specify the layout of web pages

   3. JavaScript to program the behavior of web pages
```

## HTML笔记

大致格式

```python
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>标题（title）</title>
</head>
<body>
<h1>标题（headings）</h1>
<p>段落(paragraph)</p>
</body>
</html>
```

html标签(tag), 通常是一对

```html
<tagname>content goes here...</tagname>
```

html属性(Attributes), 比如src，href，width等，style属性可以指定颜色和字体

中文和特殊字符在head标签中声明

```html
<meta charset="UTF-8">
```

链接

```html
 <a href="https://zhangjk98.xyz.com">This is a link</a>
```

图片

```html
 <img src="image.jpg" alt="图片" width="104" height="142">
```

空元素 br用来换行

meta标签用来标明一些信息，SEO的时候可能用到

```html
<meta charset="UTF-8">
<meta name="author" content="Zhang.j.k">
```

```html
 <!-- 这是一个注释（comments） -->
```

格式标签（formatting）

```html
<b> 	Defines bold text
<em> 	Defines emphasized text
<i> 	Defines italic text
<small> 	Defines smaller text
<strong> 	Defines important text
<sub> 	Defines subscripted text
<sup> 	Defines superscripted text
<ins> 	Defines inserted text
<del> 	Defines deleted text
<mark> 	Defines marked/highlighted text
```

引用标签（Quotation）

```html
<abbr> 	Defines an abbreviation or acronym
<blockquote> 	Defines a section that is quoted from another source
<q> 	Defines a short inline quotation
```

html的字符在markdown中会有渲染的问题，如果像显示为纯文本，可以引用或者使用\进行转义

### 使用CSS

有三种方式，Inline, internal, external
Inline就是style那样的加属性
internal用style标签：

```html
<style>
body {background-color: powderblue;}
h1   {color: blue;}
p    {color: red;}
</style>
```

external是独立出来一个css文件，html进行调用。

### 使用javascript

在html中使用script标签引入js进行交互和产生动态效果。

```html
<script>
document.getElementById("demo").innerHTML = "Hello JavaScript!";
</script>
```

出于安全问题，有些浏览器禁用js脚本，此时用 noscripts 标签展示相应的替代html内容。

## CSS

CSS（Cascading Style Sheets）用来指定html的格式。有selector和declaration两个部分。
像这样:

```css
/*注释这样写*/
p {color:red;text-align:center;}
```

CSS在html中的定位有选择id（#后加id）选择class属性（ . 号后加class）两种方式

## Javascript

这种格式：

```javascript
function myFunction() {
 document.getElementById("demo").innerHTML = "Paragraph changed.";
}
```

用scripts标签放到html的head或者body里面

```html
<script src="myScript.js"></script>
```

### Console控制台调试

调试的时候F12打开控制台，可以看到console输出的信息

### JSON

用JSON.stringify和JSON.parse转成字符串和还原

### 线程

javascript是用的单线程（在浏览器上）,使用异步

主要是用回调函数、事件监听来实现异步

### 变量和类

放在函数里加var就是内部变量，否则就是外部变量
类使用new来新建

```javascript
var objectSample = new Array();
```

### DOM

Document Object Model

HTML树形结构，元素被视为节点（node）

```javascript
document.getElementById()
```

这样获取元素。
节点有Document节点（作用于整个HTML）和Element节点

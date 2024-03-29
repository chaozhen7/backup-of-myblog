layout: post
title: CSS 垂直居中设置
date: 2016-1-17 18:20:22
tags: 
	- CSS
	- Web 前端
comments: true
copyright: true
---
### **1. 父元素高度确定的单行文本** ###
父元素高度确定的单行文本的竖直居中的方法是通过设置父元素的 height 和 line-height 高度一致来实现的。如下代码：
```html
<div class="container">
    hi,imooc!
</div>
```
css代码：
```css
.container{
    height:100px;
    line-height:100px;
    background:#999;
}
```

<!--more-->

### **2. 父元素高度确定的多行文本** ###
父元素高度确定的多行文本、图片、块状元素的竖直居中的方法有两种：

**方法一**
使用插入 table (包括tbody、tr、td)标签，同时设置 vertical-align：middle

说到竖直居中，css 中有一个用于竖直居中的属性 vertical-align，但这个样式只有在父元素为 td 或 th 时，才会生效。所以又要插入 table 标签了。下面看一下例子：

html代码：
```html
<body>
   <table><tbody><tr><td class="wrap">
      <div>
         <p>看我是否可以居中。</p>
         <p>看我是否可以居中。</p>
         <p>看我是否可以居中。</p>
         <p>看我是否可以居中。</p>
      </div>
   </td></tr></tbody></table>
</body>
```
css代码：
```css
table td{height:500px;background:#ccc}
```
因为 td 标签默认情况下就默认设置了 vertical-align 为 middle，所以我们不需要显式地设置了。

**方法二**
在 chrome、firefox 及 IE8 以上的浏览器下可以设置块级元素的 display 为 table-cell，激活 vertical-align 属性，但注意 IE6、7 并不支持这个样式。

html代码：
```html
<div class="container">
    <div>
        <p>看我是否可以居中。</p>
        <p>看我是否可以居中。</p>
        <p>看我是否可以居中。</p>
        <p>看我是否可以居中。</p>
        <p>看我是否可以居中。</p>
    </div>
</div>
```
css代码：
```css
.container{
    height:300px;
    background:#ccc;
    display:table-cell;/*IE8以上及Chrome、Firefox*/
    vertical-align:middle;/*IE8以上及Chrome、Firefox*/
}
```
这种方法的好处是不用添加多余的无意义的标签，但缺点也很明显，它的兼容性不是很好，不兼容 IE6、7。
 

 
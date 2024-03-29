layout: post
title: CSS 水平居中设置
date: 2016-1-17 17:20:22
tags: 
	- CSS
	- Web 前端
comments: true
toc: true
copyright: true
---
### **1. 行内元素水平居中** ###
如果被设置元素为文本、图片等行内元素时，水平居中是通过给父元素设置 `text-align:center` 来实现的。如下代码：

html代码：
```html
<body>
  <div class="txtCenter">我是文本，哈哈，我想要在父容器中水平居中显示。</div>
</body>
```
css代码：
```css
  div.txtCenter{
    text-align:center;
  }
```

<!--more-->

### **2. 定宽块状元素水平居中** ###
当被设置元素为块状元素时用 text-align：center 就不起作用了，这时也分两种情况：**定宽块状元素**和**不定宽块状元素**。下面来讲讲定宽块状元素。

满足定宽和块状两个条件的元素是可以通过设置 “左右margin” 值为 “auto” 来实现居中的。我们来看个例子就是设置 div 这个块状元素水平居中：

html代码：
```html
<body>
  <div>我是定宽块状元素，哈哈，我要水平居中显示。</div>
</body>
```
css代码：
```css
div{
    border:1px solid red;  /*为了显示居中效果明显为 div 设置了边框*/
    width:500px;   /*定宽*/
    margin:20px auto;  /* margin-left 与 margin-right 设置为 auto */
}
```
也可以写成：
```css
margin-left:auto;
margin-right:auto;
```
注意：元素的“上下 margin” 是可以随意设置的。

### **3. 不定宽块状元素水平居中** ###
在实际工作中我们会遇到需要为“不定宽度的块状元素”设置居中，比如网页上的分页导航，因为分页的数量是不确定的，所以我们不能通过设置宽度来限制它的弹性。

不定宽度的块状元素有三种方法居中（这三种方法目前使用的都比多）：
1. 加入 table 标签
2. 设置 display;inline 方法
3. 设置 position:relative 和 left:50%;

**加入 table 标签**

第一步：为需要设置的居中的元素外面加入一个 table 标签 ( 包括 &lt;tbody>、&lt;tr>、&lt;td> )。

第二步：为这个 table 设置 “左右 margin 居中”（这个和定宽块状元素的方法一样）。

举例如下：

html代码：
```html
<div>
   <table>
      <tbody>
         <tr><td>
            <ul>
               <li><a href="#">1</a></li>
               <li><a href="#">2</a></li>
               <li><a href="#">3</a></li>
            </ul>
         </td></tr>
      </tbody>
   </table>
</div>
```
css代码：
```css
table{
    margin:0 auto;
}
ul{list-style:none;margin:0;padding:0;}
li{float:left;display:inline;margin-right:8px;}
```

**设置 display;inline 方法**
第二种方法：改变块级元素的 display 为 inline 类型，然后使用 text-align:center 来实现居中效果。如下例子：

html代码：
```html
<body>
   <div class="container">
      <ul>
         <li><a href="#">1</a></li>
         <li><a href="#">2</a></li>
         <li><a href="#">3</a></li>
      </ul>
   </div>
</body>
```
css代码：

```css
.container{
    text-align:center;
}
.container ul{
    list-style:none;
    margin:0;
    padding:0;
    display:inline;
}
.container li{
    margin-right:8px;
    display:inline;
}
```
这种方法相比第一种方法的优势是不用增加无语义标签，简化了标签的嵌套深度，但也存在着一些问题：它将块状元素的 display 类型改为 inline，变成了行内元素，所以少了一些功能，比如设定长度值。

**设置 position:relative 和 left:50%**

通过给父元素设置 float，然后给父元素设置 position:relative 和 left:50%，子元素设置 position:relative 和 left:-50% 来实现水平居中。举例如下：

html代码：
```html
<body>
   <div class="container">
      <ul>
         <li><a href="#">1</a></li>
         <li><a href="#">2</a></li>
         <li><a href="#">3</a></li>
      </ul>
   </div>
</body>
```
css代码：
```css
.container{
    float:left;
    position:relative;
    left:50%
}

.container ul{
    list-style:none;
    margin:0;
    padding:0;
    
    position:relative;
    left:-50%;
}
.container li{float:left;display:inline;margin-right:8px;}
```
这种方法可以保留块状元素仍以 display:block 的形式显示，优点不添加无语议表标签，不增加嵌套深度，但它的缺点是设置了 position:relative，带来了一定的副作用。

这三种方法使用得都非常广泛，各有优缺点，具体选用哪种方法，可以视具体情况而定。
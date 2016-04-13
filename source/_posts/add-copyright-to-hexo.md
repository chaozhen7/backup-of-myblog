layout: post
title: 在 Hexo 中给文章添加版权信息
date: 2016-3-1 20:20:22
tags: 
   - Hexo
comments: true
copyright: true
---

### **step 1** ###

`layout\_partial\post\` 目录下新建 `copyright.ejs` 内如如下：

```html
<div class="article-footer-copyright">

	添加你的版权信息

</div>

```

<!--more-->

### **step 2** ###

`layout\_partial\article.ejs` 文件下添加如下代码：

```html
<% if (!index && post.copyright==true){ %>
	<%- partial('post/copyright') %>
<% } %>
```

#### **step 3** ####

`source\css\_partial\` 下新建 `copyright.styl` 内容如下：

```CSS
.article-footer-copyright{
   border-top:1px solid #d3d3d3;
   margin: 20px auto;     
   padding-left:2em;
   width: 80%;
}

.article-footer-copyright p{
  font-size:1.2em;
}

.article-footer-copyright span,.copyright abbr{
  color:#3d3d3d;
}
div.copyright{
	
     font-weight:bold;
    color:#FCB297;
       padding:0.3em 0.5em;
       margin:0 0 1em 0;
       border-bottom:none;
       background-color:#AAD2F0;
       -moz-border-radius: 1em;
       -webkit-border-radius:1em;
       border-radius: 1em;

  -moz-box-shadow:inset 0px 1px 0px 0px #eeeeee;
  -webkit-box-shadow:inset 0px 1px 0px 0px #eeeeee;
  box-shadow:inset 0px 1px 0px 0px #eeeeee;
  background:-webkit-gradient( linear, left top, left bottom, color-stop(0.05, #aad2f0), color-stop(1, #8bc1ed) );
  background:-moz-linear-gradient( center top, #aad2f0 5%, #8bc1ed 100% );
  /* filter:progid:DXImageTransform.Microsoft.gradient(startColorstr='#aad2f0', endColorstr='#8bc1ed'); */
  background-color:#aad2f0;
  border:1px solid #dcdcdc;
  text-shadow:1px 1px 0px #eeeeee;
  
}
div.article-footer-copyright{
	margin-top:2em;
	padding:1.5em;
	border:1px solid #d3d3d3;
	background-color:#DEEBF7;
}
.article-footer-copyright p{
	line-height: 160%;
	margin: 10px;
	font-size: 100%;
}
```

### **step 4** ###

`source\css\styl.styl` 文件下新添如下代码：

```css
@import '_partial/copyright'
```

### **step5 ** ###

完成！

只要在需要添加版权信息的文章中，添加 `copyright: true` 即可显示版权信息。
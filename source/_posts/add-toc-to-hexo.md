layout: post
title: 在 Hexo 中给文章添加目录
date: 2015-12-13 20:20:22
tags: 
   - Hexo
comments: true
copyright: true
---

目录的样式可以参见下面的图片。动态效果可以打开本博客下的带有目录的文章查看。
![目录样式](/img/articles/toc.jpg)

<!--more-->

下面开始正文了。
文章目录肯定是添加到 <code>post</code> 布局上，这个毋庸置疑，因为只有看文章详情页的时候才需要目录。那么我们在目录 <code>layout/_partial/post/</code> 下创建 <code>toc.ejs</code> 文件，代码如下：
``` html
<div style="width:100%;float:left;margin-bottom:25px;">
   <div id="toc" class="toc-article" >
      <div class="toc-title">
         <p>
            <span> 目录 </span>
            <a href="#" onclick="javascript:return openct(this);" title="展开">[+]</a>
         </p>
      </div>
      <%- toc(post.content, {list_number: false}) %>
   </div>
</div>
<script>
   function openct(e){    
      if (e.innerHTML == '[+]') {
         $(e).attr('title', '收起').html('[-]');
         $(".toc-article ol").show();
      } else {
         $(e).attr('title', '展开').html('[+]');
         $(".toc-article ol").hide();
      }
      e.blur();
      return false;
   }
</script>
```

这里使用了 Hexo 提供的 <code>toc()</code> 帮助函数，它的使用方法如下：
```
<%- toc(str, [options]) %>
```
<code>str</code> 就是文章内容，<code>options</code> 有两个参数，一个是 <code>class</code>，也就是html标签的class值，默认为toc；一个是 <code>list_number</code>，是否显示列表编号，默认值是true。

接下考虑把这个局部模块放到哪呢，既然属于 <code>post</code> 布局，那么就看看 <code>layout/post.ejs</code> 代码如下:
```
<%- partial('_partial/article', {item: page, index: false}) %>
```
很明显，我们要到 <code>_partial/article.ejs</code> 文件里添加 <code>toc.ejs</code>，添加后 <code>article.ejs</code> 代码如下：
```html
......
    <div class="entry">
      <% if (item.excerpt && index){ %>   
        <%- item.excerpt %>
      <% } else { if (post.toc == true) {%>  
        <%- partial('post/toc') %>
      <% } %>
        <%- item.content %>
      <% } %>
    </div>
......
```
<code>page.toc==true</code>，这一块的主要作用是只有在文章的前置声明里设置 <code>toc: true</code> 才能开启目录功能，默认是关闭的。
接下来就是设置样式了，进入 <code>source/css/_partial/</code> 目录下，创建 <code>toc.styl</code>，代码如下：
```CSS
.toc-article {
   float:left; 
   min-width:200px;
   padding: 4px 10px;
   font-size: 12px; 
   background-color: #eee;
   border: 1px solid #ccc;
}
.toc-article a {
   color: #369;
   border-bottom: 0px;
}
.toc-article ol {
  
   margin: 0px 14px 0px;
   line-height: 160%;
   padding-left: 14px;
   list-style-type: none;
   display: none;
}
.toc-article li {
   list-style: decimal;
   list-style-type: none;
   margin: 0px;
   padding: 0px;
}
.toc-article .toc-title p{
   text-align: right;
   margin: 0;
}
.toc-article .toc-title p span{
   float: left;
}
```
最后别忘了在 <code>source/css/style.styl</code> 文件里加入这句了 <code>@import '_partial/toc'</code>, 最好加在最后面。
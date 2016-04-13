wolayout: post
title: Hexo yilia主题添加网站访客人数统计
date: 2015-11-30 15:28:22
tags: 
	- Hexo
comments: true
copyright: true 
---
这里主要使用的是[amazingcounters网站](http://www.amazingcounters.com/)提供的统计功能。具体效果如下图：

![](/img/articles/addcountertohexo.jpg)
<!--more-->
下面开始介绍具体的添加流程。
## 1.获取代码
我们打开[amazingcounters网站主页](http://www.amazingcounters.com/)。首先注册账号，然后他会提示你每一步要怎么做，首先要选择的是你网站上显示访客数量的样式，根据个人喜好自己选择。一直到最后会有一个复制代码的过程，把代码复制下来进行下一步操作。
## 2.添加代码到主题中
为了方便的开启或者关闭访客数量统计功能我们可以在主题的配置文<code>_config.yml</code>里面添加一个变量，<code>counter: true</code>当改为false时即关闭了访客统计功能。  

我们在<code>\hexo\themes\yilia\layout\_partial</code>文件夹下新建一个<code>counter.ejs</code>文件，并将在前面复制到的代码粘帖在这个里面。如果想设置访客人数的显示的样式可以在这里略作修改。    

接下来打开<code>\hexo\themes\yilia\layout\_partial</code>文件夹下的<code>left-col.ejs</code>文件，找到如下代码      
```html
	<div class="overlay"> </div>
```
将下面的代码添加到这个div标签中   
```html
<% if(theme.counter) {%>
  <%- partial('counter') %>
<% } %>
```
至此，就已经添加好了统计访客并在主页显示的功能了。


## 写在后面
如果想更改在主页上显示的样式可以到amazingcounters网站上修改下配置就好了，无需更改代码。

第二步的最后在<code>left-col.ejs</code>插入代码仅仅是针对在我所给的示例的位置显示访客人数，要想在其他位置显示，需要将所给代码插入到其所对应的文件中。


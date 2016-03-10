layout: post
title: 利用 javascript 实现回到顶部效果
date: 2016-1-09 23:20:22
tags: 
	- Javascript
	- Web 前端
comments: true
---

实现回到顶部效果除了用 js 可以实现之外还可以用锚链接实现，把 a 标签的 href 的属性设置为"#"时，点击该标签就会回到顶部，但是这种方式从用户体验的角度上来讲并不是很好。这里主要介绍如何利用 js 来实现这个功能，并提供较好的用户体验。

**需求**
1. 滚动条距离顶部较远时显示返回顶部按钮，较近时不显示。
2. 鼠标未放到返回顶部按钮上时，返回按钮显示向上的箭头，鼠标放在上面时显示“返回顶部”字样。
3. 返回顶部的速度随着滚动条到顶部的距离减小而减小。
4. 再返回顶部的过程中，如果滚动鼠标滚轮可以暂停到当前位置。

<!--more-->

**主要知识点**
1. DOM 操作
document.getElementById
document.documentElement.scrollTop  滚动条到顶端的距离，可以读写（适用 IE 类浏览器）
document.body.scrollTop  滚动条到顶端的距离，可以读写（适用 chrome 类浏览器）

2. 事件运用
windown.onload  页面加载完毕后触发
onclick 点击后触发
windown.onscroll 滚动条滚动时触发

3. 定时器
setInterval()  设置定时器，需传入两个参数：第一个是重复执行的函数，第二个是函数重复执行的时间间隔
clearInterval() 关闭定时器，需传入一个参数：定时器对象

**源码**
html页面

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en">
<head>
	<meta http-equiv="Content-Type" content="text/html;charset=UTF-8" />
	<title>Javascript 返回顶部</title>
	<link rel="stylesheet" href="style.css">
	<script type="text/javascript" src="script.js"></script>
</head>
</head>
<body>
	<a href="javascript:;" id="btn" title="回到顶部"></a>  <!-- href="javascript:;" 取消标签 a 的默认效果 -->
	<div class="content">
		<!-- 网页主要内容 -->
	</div>
</body>
</html>
```

CSS 文件
```css
#btn {
	width:40px; 
	height:40px; 
	position:fixed; 
	right:65px; 
	bottom:10px; 
	display:none;   /*默认不显示*/
	background:url(images/top_bg.png) no-repeat left top; /*图片的上半部分*/
	}
#btn:hover {
	background:url(images/top_bg.png) no-repeat 0 -39px; /*图片的下半部分*/
	}
```

按钮背景图片 40*80


![返回顶部的图片，大小为40*80](/img/articles/top_bg.png)

javascript 文件
```javascript
//页面加载完毕后触发
window.onload = function(){
	var obtn = document.getElementById("btn");
	//获取页面可视区的高度
	var clientHeight = document.documentElement.clientHeight;
	var timer = null; 
	var isTop = true; //回到顶部的过程中如果滚动鼠标滚条会暂停
	
	//滚动条滚动时触发
	window.onscroll = function(){
		//获取滚动条距离顶部的高度，IE:document.documentElement.scrollTop  chrome:document.body.scrollTop
		var osTop = document.documentElement.scrollTop || document.body.scrollTop;
		if(osTop >= clientHeight){
			obtn.style.display = 'block';
		}else{
			obtn.style.display = 'none';
		}
		if(!isTop){
			clearInterval(timer);
		}
		isTop = false;
	}
	
	obtn.onclick = function(){
		
		//设置定时器
		timer = setInterval(function(){
			var osTop = document.documentElement.scrollTop || document.body.scrollTop;
			var speed = Math.floor(-osTop / 6);  //速度随距离动态变化，越来越小
			document.documentElement.scrollTop = document.body.scrollTop = osTop + speed;
			isTop = true;
			if(osTop == 0){
				clearInterval(timer); //回到顶部时关闭定时器
			}
		},30)
	}
}
```
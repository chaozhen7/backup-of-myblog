layout: post
title:  jQuery选择器
date: 2015-11-03 18:36:22
tags: 
	- jQuery
	- Web 前端
comments: true
copyright: true
---
## 1. 基础选择器
### ID选择器
例如，```$(“#id”)```
### 标签名选择器
例如，```$(“div”)```可以选取网页中的所有div元素；```$(“a”)``` 可以选择网页中所有的a元素。
<!--more-->
### 根据元素的CSS类选择
例如，使用```$(“.ClassName”)```可以选择网页中所有应用了CSS类（类名为ClassName）的html元素
### 选择所有html元素
使用```$(“*”)```可以选取网页中所有的html元素
### 同时选择多个html元素
使用```$(selector1,selector2,……,selectorN)```可以选择多个html元素
## 2. 层次选择器
### ancestordescendant（祖先 后代）选择器
ancestor descendant 选择器可以选取指定祖先元素的所有指定类型的后代元素。例如，使用 ```$(“form input”)```可以选择表单中的所有input元素。
### parent> child（父 > 子）选择器
parent > child选择器可以选取指定父元素的所有子元素，子元素必须包含在父元素中。例如，使用```$(“form > input”)```可以选择表单中的所有input元素。
### prev+ next（前 + 后）选择器
prev + next选择器可以选取紧接在指定的prev元素后面的next元素。例如，使用```$(“labe + input ”)```可以选择所有紧接在label元素后面的input元素
### prev~ siblings（前 ~ 兄弟）选择器
prev ~ siblings选择器可以选取指定的prev元素后面的根据 siblings 过滤的元素。例如，使用 ```$(“#prev ~ div”)``` 可以选择所有紧接在id为prev 的元素后面的 div 元素。
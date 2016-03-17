layout: post
title: python中的爬虫神器 XPath 介绍
date: 2016-3-1 10:20:22
tags: 
	- 爬虫
	- Python
category: Python
comments: true
toc: true
---

XPath即为XML路径语言，它是一种用来确定XML（标准通用标记语言的子集）文档中某部分位置的语言。XPath基于XML的树状结构，提供在数据结构树中找寻节点的能力。 xPath 同样也支持HTML.

XPath 是一门小型的查询语言，这里我们将它与 python 爬虫相结合来介绍。

<!--more-->

## **1. python 中如何安装使用XPath** ##

step1: 安装 lxml 库。

step2:　from lxml import etree

step3: Selector = etree.HTML(网页源代码)

step4: Selector.xpath(一段神奇的符号)

## **2. XPath 语法** ##

XPath 使用路径表达式来选取 XML 文档中的节点或节点集。节点是通过沿着路径 (path) 或者步 (steps) 来选取的。

XPath 使用路径表达式在 XML 文档中选取节点。节点是通过沿着路径或者 step 来选取的。

假设有如下的文档：

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<bookstore>
   <book>
      <title lang="eng">Harry Potter</title>
      <price>29.99</price>
   </book>
   <book>
      <title lang="eng">Learning XML</title>
      <price>39.95</price>
   </book>
</bookstore>
```

下面列出了最有用的路径表达式：

|表达式|	描述|
|:--:|:--|
|nodename	|选取此节点的所有子节点|
|/	|从根节点选取|
|//	|从匹配选择的当前节点选择文档中的节点，而不考虑它们的位置|
|.	|选取当前节点|
|..	|选取当前节点的父节点|
|@	|选取属性|

在下面的表格中，我们已列出了一些路径表达式以及表达式的结果：

|路径表达式|	结果|
|:--:|:--|
|bookstore	| 选取 bookstore 元素的所有子节点|
|/bookstore	|选取根元素 bookstore。</br>注释：假如路径起始于正斜杠( / )，则此路径始终代表到某元素的绝对路径！|
|bookstore/book|	选取属于 bookstore 的子元素的所有 book 元素|
|//book	| 选取所有 book 子元素，而不管它们在文档中的位置|
|bookstore//book |	选择属于 bookstore 元素的后代的所有 book 元素，而不管它们位于 bookstore 之下的什么位置|
|//@lang	| 选取名为 lang 的所有属性|

**
鉴于篇幅的，只列出了 xpath 的部分语法，更多实例，可以参考 [w3chool](http://www.w3school.com.cn/xpath/xpath_syntax.asp) 的资料**

## **3. python 中的使用示例** ##

假设有如下 html 源码：
```html
<div id="content">   
   <ul id="useful">
      <li>有效信息1</li>
      <li>有效信息2</li>
      <li>有效信息3</li>
   </ul>
   <ul id="useless">
      <li>无效信息1</li>
      <li>无效信息2</li>
      <li>无效信息3</li>
   </ul>
</div>
<div id="url">
   <a href="http://cighao.com">陈浩的博客</a>
   <a href="http://cighao.com.photo" title="陈浩的相册">点我打开</a>
</div>
```

我们假设把上面的源码已经赋值给了字符串变量 `html`:

```
from lxml import etree

# 假设已经存在 html 变量，值为上面的源码

selector = etree.HTML(html)

# 提取 li 中的有效信息123
content = selector.xpath('//ul[@id="useful"]/li/text()')
for each in content:
    print(each)

#提取 a 中的属性
link = selector.xpath('//a/@href')
for each in link:
    print(each)

title = selector.xpath('//a/@title')
for each in title:
    print(each)

```
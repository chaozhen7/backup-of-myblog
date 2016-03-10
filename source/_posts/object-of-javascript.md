layout: post
title: JavaScript之DOM对象和内置对象
date: 2015-10-22 21:47:22
tags: 
	- Javascript
	- Web 前端
comments: true
toc: true
---
## 1. window 和document 对象
浏览器每次加载和显示页面时，都在内存里创建页面及其全部元素的一个内部表示体系，也就是DOM。在DOM 里，页面的元素具有一个逻辑化、层级化的结构，就像一个由父对象和子对象组成的树形结构。这些对象及其相互关系构成了Web 页面及显示页面的浏览器的抽象模型。每个对象都有“属性”列表来描述它，而利用JavaScript可以使用一些方法来操作这些属性。

这个层级树的最顶端是浏览器window对象，它是DOM 树里一切对象的根。window对象具有一些子对象，如下图所示。图中第一个子对象是document，这也是最经常使用的对象。浏览器加载的任何HTML页面都会创建一个document对象，包含全部HTML内容及其他构成页面显示的资源。利用JavaScript以父子对象的形式就可以访问这些信息。这些对象都具有自己的属性和方法。
![DOM对象](/img/articles/objectofjavascript.jpg) 
<!--more-->
## 2. DOM对象和内置对象
### 2.1 alert()
alert() 没有返回值，向用户弹出一个对话框。
### 2.2 confirm()
confirm() 对话框为用户提供了一个选择，可以点击“确定”或者“取消”按钮。点击任意一个按钮都会关闭对话框，让脚本继续执行，但根据哪个按钮被单击，confirm()方法返回不同的值。单击“确定”返回布尔值“真”，单击“取消”返回布尔值“假”。
### 2.3 prompt()
prompt() 是打开模态对话框的另一种方式，它允许用户输入信息。使用示例：
``` javascript
var name =prompt(“what is your name?”);
```
prompt() 方法还可以有第二个可选参数，表示默认的输入内容，从而避免用户直接点击“确定”而不输入任何内容。如，
``` javascript
var name = prompt(“what is your name?”,”John”);
```
当用户输入了信息，然后点击“确定”按钮或者按回车键后，返回值就是用户输入的字符串；如果用户没有在对话框输入信息就点击“确定按钮”或者按回车键，返回值就是prompt()方法设置的第二个可选参数的值（如果有的话）；当用户简单关闭了这个对话框（点击了“取消”或者按了Esc键），返回值就是null。
### 2.4 根据ID选取元素
如果想从HTML页面里选择某个特定id的元素，我们只需要把相应的id作为参数来调用document对象的 getElementById()方法，他就会返回特定id的页面元素所对应的DOM对象。例如，
``` javascript
var myDiv = document.getElementById(“div1”);
```
注：innerHTML属性的介绍：innerHTML属性可以读取或者设置特定页面元素内部的HTML内容。假设假设HTML页面包含如下元素：
``` html
<div id=”div1”>
     <p> Here is some originaltext. </p>
</div>
```
如果执行下面的语句
``` javascript
var myDivContents =document.getElementById().innerHTML;
```
那么myDivContents则包含了如下字符串：Hereis some original text.
### 2.5 访问浏览器记录
在JavaScript里，浏览器的历史记录是以 window.history对象代表的，它基本上就是访问过的URL列表。这个对象的方法让我们能够使用这个列表，但是不能直接修改这些URL。这个对象只有一个属性，就是他的长度，即history.length。有三个方法，forward()，backward()，go()。
- forward()和backward()方法相当于点击浏览器的“前进”和“后退”按钮。
- go()方法有一个参数，正数或者负数，可以跳到历史记录列表里的相对位置。如，
``` javascript
history.go(-3);  // 回退3个页面
history.go(2);  // 前进2个页面
```
这个方法也可以接受字符串作为参数，找到历史记录列表里第一匹配的URL。如，
``` javascript
history.go(“example.com”);  //到达历史记录列表里面第一个包含“example.com”的URL
```
### 2.6 location对象
location对象主要是包含当前加载页面的URL信息。
location对象的属性：
- hash: 设置或返回从井号 (#) 开始的 URL（锚）
- host: 设置或返回主机名和当前 URL 的端口号 
- hostname: 置或返回当前 URL 的主机名 
- href: 设置或返回完整的 URL 
- patdname: 设置或返回当前 URL 的路径部分 
- port: 设置或返回当前 URL 的端口号 
- protocol: 设置或返回当前 URL 的协议 
- search: 设置或返回从问号 (?) 开始的 URL（查询部分）>
利用location对象导航。第一种是直接设置对象的href属性：
``` javascript
location.href= ‘www.baidu.com’;
```
使用这种方法把用户转移到新页面时，原始页面还保留在浏览器的历史记录里面。第二种是直接用新的URL代替当前页面，即把当前页面从历史记录列表里删除。如：
``` javascript
location.replace(‘www.baidu.com’);
```
更多内容可以参考  [JavaScript手册](http://download.csdn.net/detail/cighao/9206313)
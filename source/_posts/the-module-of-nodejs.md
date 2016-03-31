layout: post
title: nodejs 中模块使用的介绍
date: 2016-1-10 9:20:22
tags: 
	- Nodejs
comments: true
copyright: true
---
**模块的分类**
核心模块： 如 http
文件模块： 如 var util = require('./util.js')
第三方模块： 如 var promise=require('bluebird')

**模块的使用流程**
创建模块： teacher.js
导出模块： exports.add = function(){}
加载模块： var teacher = require('./teacher.js')
使用模块： teacher.add('Tom')

<!--more-->

**require,exports 和 module**
*require* 函数用于在当前模块中加载和使用别的模块，传入一个模块名，返回一个模块导出对象。模块名可使用相对路径（以./开头），或者是绝对路径（以/ 或C: 之类的盘符开头）。另外，模块名中的.js 扩展名可以省略。

*exports* 对象是当前模块的导出对象，用于导出模块公有方法和属性。别的模块通过 require 函数使用当前模块时得到的就是当前模块的 exports 对象。

*module* 对象可以访问到当前模块的一些相关信息，但最多的用途是替换当前模块的导出对象。例如模块导出对象默认是一个普通对象。

**模块初始化**
一个模块中的 JS 代码仅在模块第一次被使用时执行一次，并在执行过程中初始化模块的导出对象。之后，缓存起来的导出对象被重复利用。

**使用举例**
我们假设有一个学校，学校里有班级，老师，学生等成员。我们可以把班级，老师，学生先模块化再整合在一起。

老师模块： teacher.js
```javascript
function add(teacher){
	console.log('Add Teacher',teacher)
}
exports.add = add;
```

学生模块： student.js
```javascript
function add(student){
	console.log('Add Student',student)
}
exports.add = add;
```

班级模块： class.js
```javascript
var student = require('./student');
var teacher = require('./teacher');
function add(teacherName,students){ //班级由一个老师和多个学生组成
	teacher.add(teacherName);	
	students.forEach(function(item,index){ 
		student.add(item);
	})
} 
exports.add=add;   //使模块成为传统的模块实例

//module.exports=add;  //使模块成为特别的对象类型
```

新建一个测试文件：index.js
```javascript
var c = require('./class');

c.add('Scot',['高富帅','白富美'])
```
这时候再命令行下输入: `node index` 就会打印输出如下信息：
> add Teacher Scot
> add Student 高富帅
> add Student 白富美

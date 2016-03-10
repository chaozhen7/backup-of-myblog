layout: post
title: JavaScript之日期和时间(date对象)
date: 2015-10-24 20:48:22
tags: 
	- Javascript
	- Web 前端
comments: true
toc: true
---
## 1. 创建具有当前日期和时间的Date对象
在JavaScript时需要使用日期和时间时，需要先创建一个date对象，代码如下，
```javascript
var myDate= new Date();
```
变量myDate就是一个Date对象，包含了创建对象时的日期和时间信息。JavaScript提供了很多方法用于获取、设置和编辑Date对象里的数据。关于Date对象的方法可以看本文的最后。
<!--more-->
## 2. 创建具有指定日期和时间的Date对象
给Date()语句传递相应的参数，就可以创建任意指定日期和时间的Date对象，方式有：
```javascript
new Date(milliseconds)   // 自1970年1月1日起的毫秒数
new Date(dateString)
new Date(year,month,day,hours,minutes,seconds,milliseconds)
```
范例：
```javascript
var d1=new Date(“October 22,1995 10:57:22”);
```
在使用分散的各部分参数时，位置靠后的参数是可选的，不明确指定的参数值是0：
```javascript
var d2=new Date(95,9,22) ;   // 1995年10月22日 00:00:00
var d2=new Date(95,9,22,10,57,0) ;   // 1995年10月22日 10:57:00
```

## 3. 设置和编辑日期与时间

设置和编辑日期与时间的函数也比较多，在最后的表格会给出。需要注意的是，比如当我们修改date中的day（某月的一天）时，其对应的星期几也会自动调整。如果我们将日期往后加个10天，其对应的月份和星期几也会自动调整。

## 4. Date对象的方法



| 方法 | 描述 |
| -----|----|
|  Date() |   返回当日的日期和时间 |   
|  getDate() |   从 Date 对象返回一个月中的某一天 (1 ~ 31) |   
|  getDay() |   从 Date 对象返回一周中的某一天 (0 ~ 6) |   
|  getMonth() |   从 Date 对象返回月份 (0 ~ 11) |   
|  getFullYear() |   从 Date 对象以四位数字返回年份 |   
|  getYear() |   请使用 getFullYear()方法代替 |   
|  getHours() |   返回 Date 对象的小时 (0 ~ 23) |   
|  getMinutes() |   返回 Date 对象的分钟 (0 ~ 59) |   
|  getSeconds() |   返回 Date 对象的秒数 (0 ~ 59) |   
|  getMilliseconds() |   返回 Date 对象的毫秒(0 ~ 999) |   
|  getTime() |   返回 1970 年 1 月 1 日至今的毫秒数 |   
|  getTimezoneOffset() |   返回本地时间与格林威治标准时间 (GMT) 的分钟差 |   
|  getUTCDate() |   根据世界时从 Date 对象返回月中的一天 (1 ~ 31) |   
|  getUTCDay() |   根据世界时从 Date 对象返回周中的一天 (0 ~ 6) |   
|  getUTCMonth() |   根据世界时从 Date 对象返回月份 (0 ~ 11) |   
|  getUTCFullYear() |   根据世界时从 Date 对象返回四位数的年份 |   
|  getUTCHours() |   根据世界时返回 Date 对象的小时 (0 ~ 23) |   
|  getUTCMinutes() |   根据世界时返回 Date 对象的分钟 (0 ~ 59) |   
|  getUTCSeconds() |   根据世界时返回 Date 对象的秒钟 (0 ~ 59) |   
|  getUTCMilliseconds() |   根据世界时返回 Date 对象的毫秒(0 ~ 999) |   
|  parse() |   返回1970年1月1日午夜到指定日期（字符串）的毫秒数 |   
|  setDate() |   设置 Date 对象中月的某一天 (1 ~ 31) |   
|  setMonth() |   设置 Date 对象中月份 (0 ~ 11) |   
|  setFullYear() |   设置 Date 对象中的年份（四位数字） |   
|  setYear() |   请使用 setFullYear()方法代替 |   
|  setHours() |   设置 Date 对象中的小时 (0 ~ 23) |   
|  setMinutes() |   设置 Date 对象中的分钟 (0 ~ 59) |   
|  setSeconds() |   设置 Date 对象中的秒钟 (0 ~ 59) |   
|  setMilliseconds() |   设置 Date 对象中的毫秒 (0 ~ 999) |   
|  setTime() |   以毫秒设置 Date 对象 |   
|  setUTCDate() |   根据世界时设置 Date 对象中月份的一天 (1 ~ 31) |   
|  setUTCMonth() |   根据世界时设置 Date 对象中的月份 (0 ~ 11) |   
|  setUTCFullYear() |   根据世界时设置 Date 对象中的年份（四位数字） |   
|  setUTCHours() |   根据世界时设置 Date 对象中的小时 (0 ~ 23) |   
|  setUTCMinutes() |   根据世界时设置 Date 对象中的分钟 (0 ~ 59) |   
|  setUTCSeconds() |   根据世界时设置 Date 对象中的秒钟 (0 ~ 59) |   
|  setUTCMilliseconds() |   根据世界时设置 Date 对象中的毫秒 (0 ~ 999) |   
|  toSource() |   返回该对象的源代码 |   
|  toString() |   把 Date 对象转换为字符串 |   
|  toTimeString() |   把 Date 对象的时间部分转换为字符串 |   
|  toDateString() |   把 Date 对象的日期部分转换为字符串 |   
|  toGMTString() |   请使用 toUTCString()方法代替 |   
|  toUTCString() |   根据世界时，把 Date 对象转换为字符串 |   
|  toLocaleString() |   根据本地时间格式，把 Date 对象转换为字符串 |   
|  toLocaleTimeString() |   根据本地时间格式，把 Date 对象的时间部分转换为字符串 |   
|  toLocaleDateString() |   根据本地时间格式，把 Date 对象的日期部分转换为字符串 |   
|  UTC() |   根据世界时返回 1970 年 1 月 1 日 到指定日期的毫秒数 |   
|  valueOf() |   返回 Date 对象的原始值 |   
|  valueOf() |   返回 Date 对象的原始值 |    
 

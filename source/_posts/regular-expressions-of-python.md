layout: post
title: python 正则表达式的使用
date: 2015-8-22 13:43:22
tags: 
	- Python
	- 正则表达式
	- 爬虫
comments: true
toc: true
---
## 1. 常用符号  
. ：匹配任意字符，换行符 \n 除外  
\* ：匹配前一个字符0次或无限次  
? ：匹配前一个字符0次或1次  
.\* ：贪心算法，尽可能的匹配多的字符  
.\*? ：非贪心算法  
() ：括号内的数据作为结果返回  
<!--more-->
## 2. 常用方法  
findall：匹配所有符合规律的内容，返回包含结果的列表  
Search：匹配并提取第一个符合规律的内容，返回一个正则表达式对象  
Sub：替换符合规律的内容，返回替换后的值  
## 3. 使用示例  
### 3.1 . 的使用举例，匹配任意字符，换行符 \n 除外
``` python
	import re      #导入re库文件  
	a = 'xy123'    
	b = re.findall('x..',a)    
	print b    
```
打印的结果为：['xy1'] ，每个 . 表示一个占位符  
### 3.2 \* 的使用举例，匹配前一个字符0次或无限次
``` python
	a = 'xyxy123'
	b = re.findall('x*',a)
	print b
```
打印的结果为：['x', '', 'x', '', '', '', '', '']     
### 3.3 ?的使用举例，匹配前一个字符0次或1次
``` python
a = 'xy123'
b = re.findall('x?',a)
print b  
```
打印的结果为：['x', '', '', '', '', ''] 
### 3.4 .\*的使用举例
``` python
secret_code = 'hadkfalifexxIxxfasdjifja134xxlovexx23345sdfxxyouxx8dfse'
b = re.findall('xx.*xx',secret_code)  
print b 
```
打印的结果为：['xxIxxfasdjifja134xxlovexx23345sdfxxyouxx']
### 3.5 .\*？的使用举例
``` python
secret_code = 'hadkfalifexxIxxfasdjifja134xxlovexx23345sdfxxyouxx8dfse'
c = re.findall('xx.*?xx',secret_code)
print c
```
打印的结果为：['xxIxx', 'xxlovexx', 'xxyouxx']
### 3.6 ()的使用举例
``` python
secret_code = 'hadkfalifexxIxxfasdjifja134xxlovexx23345sdfxxyouxx8dfse'
d = re.findall('xx(.*?)xx',secret_code)
print d
```
打印的结果为：['I', 'love', 'you']，括号内的数据作为返回的结果
### 3.7 re.S的使用举例
``` python
s = '''sdfxxhello
xxfsdfxxworldxxasdf'''
d = re.findall('xx(.*?)xx',s,re.S)  
print d
```
打印的结果为：['hello\n', 'world']  ，re.S的作用是使 . 在匹配时包括 \n      
### 3.8 findall的使用举例
``` python
s2 = 'asdfxxIxx123xxlovexxdfd'
f2 = re.findall('xx(.*?)xx123xx(.*?)xx',s2)
print f2[0][1]
```
打印的结果为：love  
这时f2为含有一个元组的列表，该元组包含两个元素，该元组中的两个元素为两个()匹配到的内容，如果s2包含多个'xx(.*?)xx123xx(.*?)xx'这样的子串，则f2包含多个元组
### 3.9 search的使用举例
``` python
s2 = 'asdfxxIxx123xxlovexxdfd'
f = re.search('xx(.*?)xx123xx(.*?)xx',s2).group(2)
print f
```
打印的结果为：love
.group(2) 表示返回第二个括号匹配到的内容，如果是 .group(1), 则打印的就是：I
### 3.10 sub的使用举例
``` python
s = '123rrrrr123'
output = re.sub('123(.*?)123','123%d123'%789,s)
print output
```
打印的结果为：123789123  
其中的%d类似于C语言中的%d，如果 output=re.sub('123(.*?)123','123789123',s)，输出结果也为：123789123
### 3.11 \d 的使用举例，用于匹配数字
``` python
a = 'asdfasf1234567fasd555fas'
b = re.findall('(\d+)',a)
print b
```
打印的结果为：['1234567', '555'], \d+  可以匹配数字字符串
layout: post
title: disksim with ssdmodel 源码解析（二十）：统计结果的提取
date: 2016-4-12 15:23:22
tags: 
	- SSD
	- Disksim
comments: true
copyright: true
---


disksim 模拟器的输出结果是保存在一个输出文件中的，输出文件中包含很多信息，有几万行，但是我们关注的信息是比较少的，如果每次查看输出结果的时候都要去从几万行信息里面找是很麻烦的，所以可以用 python 来写个脚本，利用正则表达式来找出我们关注的信息。

<!--more-->

下面以提取平均响应时间为例来介绍（提取其他信息也类似）。

假设我们有一组 trace 数据，输出了一组模拟结果，我们要从这一组模拟结果中找出每个 trace 的平均响应时间。

平均响应时间在输出文件中是按如下方式显示的：
> ssd Response time average: 	6.394276

我们可以把输出文件从头到尾一行一行的遍历一遍，然后按照 `ssd Response time average:(.*?)` 的方式匹配出平均响应时间。

但是，在输出文件中还有一个地方出现了 "ssd Response time average:"，原文如下：
> System logorg #0 ssd Response time average: 	6.394276

所以我们在匹配的时候需要注意是在每一行的开始处匹配。

python 代码如下：

```python
# -*- coding:UTF-8 -*-
import re

# trace file name
file_array=["hm_0.detect","hm_1.detect","prn_0.detect","prxy_0.detect",
			"rsrch.detect","src1_2.detect","src2_0.detect",
			"stg_0.detect","usr_0.detect","wdev_0.detect"]

for name in file_array:
	filename = name + ".result"  # output file name
	result_file = open(filename,'r')
	for line in result_file:
		# get the response time average
		pat = "ssd Response time average:(.*?)\n"
		r = re.match(pat,line,flags=0)
		if(r!=None):
			print name + ":  " + r.group(1).strip()
	result_file.close()
```


### **re.match()函数介绍** ###

`re.match()` 尝试从字符串的**起始位置**匹配一个模式，如果不是起始位置匹配成功的话，match()就返回none。

函数语法：`re.match(pattern, string, flags=0)`

函数参数说明：

|参数	|  描述  |
|:----:|-----|
| pattern  | 	匹配的正则表达式  |
| string   |	 要匹配的字符串  |
|flags	|  标志位，用于控制正则表达式的匹配方式，如：是否区分大小写，多行匹配等等|

匹配成功re.match方法返回一个匹配的对象，否则返回None。

我们可以使用 group(num) 或 groups() 匹配对象函数来获取匹配表达式。

|匹配对象方法	|描述|
|:----:|-----|
|group(num=0)|	匹配的整个表达式的字符串，group() 可以一次输入多个组号，在这种情况下它将返回一个包含那些组所对应值的元组。|
|groups()	|返回一个包含所有小组字符串的元组，从 1 到 所含的小组号。|


**re.match与re.search的区别：**re.match 只匹配字符串的开始，如果字符串开始不符合正则表达式，则匹配失败，函数返回None；而re.search 匹配整个字符串，直到找到一个匹配。


**strip()函数：**用于移除字符串头尾指定的字符（默认为空格）。

更多关于 python 正则表达式的信息[请戳这里](../../../../2015/08/22/regular-expressions-of-python/)

layout: post
title: disksim with ssdmodel 源码解析（十）：GC分析(一)
date: 2016-3-19 15:23:22
tags: 
	- SSD
	- Disksim
comments: true  
---

垃圾回收(gc)是 SSD 中很重要的一个过程，在 ssdmodel 模块中进行垃圾回收，是在 `ssd_activate_elem()` 函数中触发的。每次有请求过来时，都会进行垃圾回收的判断，当满足一定的要求时就会进行垃圾回收，垃圾回收结束之后，会退出该函数，并不会继续处理进来的请求，请求会放在下一次来完成，如果没有达到垃圾回收的条件，那么会响应该请求。整个流程如下：

<!--more-->

![](/img/articles/disksim/gc1.jpg)

这是一个简化的流程图，实际过程比这个较为复杂。

`media is busy?` 是通过 `elem->media` 是否等于 true 来判断的。

`cleanning is background？` 是属于 SSD 的一个属性，在配置文件中进行设置。是否允许后台进行垃圾回收主要的区别在于，如果允许后台垃圾回收，不管该 element 上有没有请求都会触发程序去检查是否需要垃圾回收，然后决定是否回收。如果不允许后台垃圾回收，只有当该 element 上有请求的时候才会触发程序去进行垃圾回收的检查。

`reqs_waiting` 是通过 `elem->metadata.reqs_waiting` 来获得的，表示该 elements 是否有请求正在等待响应。

关于是否满足垃圾回收的条件，在实际的程序处理中是根据 SSD 的 copy_back (how active pages are assigned in an element) 属性来分别进行判断的。所以上面的图可以进行如下的细分。

![](/img/articles/disksim/gc2.jpg)
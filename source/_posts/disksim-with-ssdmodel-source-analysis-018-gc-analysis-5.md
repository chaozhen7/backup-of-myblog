layout: post
title: disksim with ssdmodel 源码解析（十八）：GC 分析(五)
date: 2016-3-30 20:23:22
tags: 
	- SSD
	- Disksim
comments: true
copyright: true
---


前面介绍了在没有 copyback 情况下利用贪心算法来实现 GC。今天来介绍一下在没有 copyback 的情况下利用随机算法来实现 GC 操作。

接着理一理程序的流程：

`ssd_activate_elem()` ---> `ssd_invoke_element_cleaning()` ---> `_ssd_invoke_element_cleaning()` ---> `ssd_clean_element()` ---> `(if no copyback)` ---> `ssd_clean_element_no_copyback()` ---> `(if达到gc条件)` ---> `(if随机算法)` ---> `ssd_clean_blocks_random()`


<!--more-->

今天就主要介绍 `ssd_clean_blocks_random(int plane_num, int elem_num, ssd_t *s)` 中是如何利用随机算法来进行垃圾回收的。如果这里的 plane_num=-1 则表示把该 element 当成一个整体来进行垃圾回收，否则的话就是把这个 element 上编号为 plane_num 的 plane 当成一个整体进行回收。

随机算法实在是太简单了，就直接上图吧。

![垃圾回收的随机算法](/img/articles/disksim/gc4.jpg)
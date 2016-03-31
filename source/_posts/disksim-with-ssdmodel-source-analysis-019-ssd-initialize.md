layout: post
title: disksim with ssdmodel 源码解析（十九）：ssd 的初始化
date: 2016-3-31 13:23:22
tags: 
	- SSD
	- Disksim
comments: true
copyright: true
---


ssd 的初始化是在 `ssd_init.c` 这个文件中完成的。


入口是在 `ssd_initialize()` 这个函数这里。过程如下：

<!--more-->


## step1. 初始化 diskinfo ##

调用 `ssd_initialize_diskinfo ()` 函数将 ssd 模块集成到 disk 上，主要是设置 `disksim->ssdinfo` 相关的信息。

## step2. 设置队列的回调函数 ##

调用 `ssd_setcallbacks()`，再其内部又会调用 `ioqueue_setcallbacks()`，其实代码只有一行：
```c
disksim->timerfunc_ioqueue = ioqueue_idledetected;
```
`ioqueue_idledetected` 是一个函数指针，这里主要是设置一个回调函数。


## step3. 依次初始化每一个 SSD ##

这一步主要是一个for循环，从第一个 ssd 依次到最后一个 ssd，ssd 的编号从0开始依次递增。对于每一个 ssd 的初始化可以分为以下几个步骤：

(1) 调用 `ssd_alloc_queues()` 给 ssd 队列分配内层。主要是给 ssd 的 `gang_meta.queue` 和 `element.queue` 分配内存。

(2) 调用 `ssd_verify_parameters()` 验证 ssd 的相关参数是否设置正确。ssd 最少空闲块的比例应该小于预留块的比例。如果在每个 plane 上设置独立的地址映射(即currdisk->params.alloc_pool_logic=2)那么必须要设置copyback，因为这样才能保证垃圾回收是在一个plane上进行。还有一个就是写策略的验证，ssdmodel已经不支持第一种的简单写策略了，也就是 ssd 的 write_policy=1 是不合法的了。

(3) 设置当前 ssd 的设备号，设备号是从0依次递增的，只要在循环内设置 `currdisk->devno=i` 即可。

(4) 设置当前 ssd 拥有的总的块数，也就是设置 currdisk->numblocks，这里的总的块数中的“块”不是指 ssd 上的“块”，而是指原来的磁盘上的那个“块”，也就是扇区(sector,512B)。所以这个值的单位是扇区(sector,512B)。

(5) 初始化当前 ssd 上的每个 gang。

(6) 初始化当前 ssd 上的每一个 element。这一步主要是做了两个工作，第一个是初始化当前 element 上所有的plane，第二个是初始化当前 element 上的元数据(只有写策略是第二种时才会执行此步骤，但是已经不支持第一种写策略了所以这一步必然会执行。)

 

关于 element 元数据的初始化比较复杂，后面单独介绍。
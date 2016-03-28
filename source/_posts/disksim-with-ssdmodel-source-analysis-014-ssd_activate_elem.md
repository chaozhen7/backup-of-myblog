layout: post
title: disksim with ssdmodel 源码解析（十四）：ssd_activate_elem函数分析
date: 2016-3-24 20:23:22
tags: 
	- SSD
	- Disksim
comments: true  
---



当一个请求经过一系列的处理后最终会发送到 element 上去，在 `ssd_media_access_request_element()` 函数中会把一个请求拆分成以 page 为大小的小请求，然后把这些小请求加入到 element 队列中，然后调用 `ssd_activate_elem()` 这个函数让 element 处理请求。下面主要分析 `ssd_activate_elem()` 这个函数是怎么处理请求的。

关于 GC 操作的判断也是在这个函数中触发，这个函数中关于GC部分的内容前面已经介绍过了，不再重复。**当没有进行 GC 操作且该 element 的请求队列中有请求时，会进行请求的处理**。

<!--more-->

首先介绍下这个函数中局部变量的含义：
- `read_reqs[]`：读请求的集合，保存的是 ssd_req 的指针。
- `write_reqs[]`：写请求的集合，保存的是 ssd_req 的指针。
- `max_req：SSD`: 一次能处理的最多的请求数量。如果SSD没有copyback功能 或者 num_parunits == 1，那么max_req=1，否则 max_reqs = MAX_REQS_ELEM_QUEUE(预先设定的一个值)。
- `read_total`：读请求的数量。
- `write_total`: 写请求的数量。


### **step 1: 从队列中批量取出请求** ###

不停的利用  `ioqueue_get_next_request(elem->queue)` 这个函数从 element 的队列中取请求出来，根据它是读请求还是写请求，将它放到 read_reqs[] 或者 write_reqs[] 中去，同时使 read_total 或 write_total 自增。当 element 队列中没有请求或者 read_total 和 write_total 中有一个大于 `max_req` 时，则停止从 element 的队列中取请求。

注意，如果从element中取出的某个请求已经存在 read_reqs[] 或 write_reqs[] (也就是说有重复的请求)。那么后面取出来的请求会被抛弃：将请的类型设置为 `DEVICE_ACCESS_COMPLETE` 然后加入到 intq 队列中去。判断请求是否存在数组中是通过 `ssd_already_present()` 这个函数来判断的，如果这个两个请求的 blkno 和 flags 是一样的，那么这两个请求就是一样的。


### **step 2: 处理读请求** ###

step2 和 step3 互换也没有关系。

如果 read_total>0 那么说明有读请求，这个时候可以处理读请求。没有读请求的话，这一步啥都不用处理。

处理读请求的过程中首先调用  `ssd_compute_access_time(currdisk, elem_num, read_reqs, read_total)` 这个函数，将请求全部发送过去让它来处理，处理完成后会在每个 `req->acctime` 和 `req->schtime` 中保存这个请求的时间开销。

当请求处理完成之后，会依次遍历 read_reqs[] 这个数组，依次更新每个请求的时间，还有一些其他关于时间更新方面的操作，最后将每个请求的类型更新为 `DEVICE_ACCESS_COMPLETE`，然后将其添加到 `intq` 队列中。


### **step 3: 写请求的处理** ###

和读请求的处理流程一样。先判断 write_total 是否大于 0 ，如果是的话调用  ssd_compute_access_time() 处理请求。然后依次遍历每个请求，更新一些时间相关的参数后，将每个请求的类型更新为 DEVICE_ACCESS_COMPLETE，然后将其添加到 intq 队列中。


### **step 4： 更新 element 参数** ###

主要是更新 element 的 `tot_reqs_issued` 和 `tot_time_taken` 这两个参数。





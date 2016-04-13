layout: post
title: disksim with ssdmodel 源码解析（十五）：GC 分析(二)
date: 2016-3-27 20:23:22
tags: 
	- SSD
	- Disksim
comments: true
copyright: true
---

今天主要介绍后台 GC 操作与普通 GC 操作的区别。

普通 GC 是指当一个写请求到来时，如果这个 element 上没有空闲的 page 了或者空闲的 page 到达了预先设定的下限，那么就会触发相应的 GC 操作等 GC 结束后才会进行写入操作。很明显这是会影响响应时间的。

后台 GC 是指当 SSD 处于空闲状态时，来检查 SSD 是否是需要进行 GC，如果不需要就不用进行 GC 了，如果要的话就进行 GC，由于这个时候 SSD 是处于空闲状态，这个时候进行 GC 操作就不会影响 SSD 的响应时间了。

<!--more-->

在 ssdmodel 中，普通 GC 与后台 GC 大致有两点不同：(1)普通 GC 只有在这个 element 上有请求在排队时，才会检查是否要去GC。对于后台 GC 不管这个 element 有没有请求都会去检查是否要 GC 。(2)普通 GC 只有当请求来的时候才会去触发，也就是说只有当需要写且没有地方写的时候才会触发GC，如果是后台 GC 的话，那么完成一次写请求之后，都会去检查这次的写是不是把SSD写“满”了,如果是就触发 GC，而不用等下一个写来了且写不下去了才触发 GC。

## 代码实现 ##

前面介绍了 GC 操作是在 `ssd_activate_elem(ssd_t *currdisk, int elem_num)` 函数中进行的，进入该函数后首先是判断该 element 是否处于 busy 状态，如果是的话就直接返回了。

进入到 `ssd_activate_elem()` 中有两种途径，第一种是在 `ssd_media_access_request_element()` 这个函数，这个函数主要是将一个请求分成几个以 page 为大小的小请求，然后将该请求加入到相应的 element 的队列中，然后调用 `ssd_activate_elem()` 去执行这个请求。

第二种途径是在 `ssd_access_complete_element()` 这个函数中，每完成一个 element 上的请求时都会都会在这个函数中调用了 `ssd_activate_elem()`。


`ssd_activate_elem()` 函数中 gc 部分的代码如下：

```C
if (elem->media_busy == TRUE) {
	return;
}
ASSERT(ioqueue_get_reqoutstanding(elem->queue) == 0);
// we can invoke cleaning in the background whether there is request waiting or not
if (currdisk->params.cleaning_in_background) {
	if (ssd_invoke_element_cleaning(elem_num, currdisk)) {
		return;
	}
}
ASSERT(elem->metadata.reqs_waiting == ioqueue_get_number_in_queue(elem->queue));
if (elem->metadata.reqs_waiting > 0) {
// invoke cleaning in foreground when there are requests waiting
if (!currdisk->params.cleaning_in_background) {
	if (ssd_invoke_element_cleaning(elem_num, currdisk)) {
		return;
	}
}
```

## 代码分析 ##

如果从 `ssd_media_access_request_element()` 函数中调用 `ssd_activate_elem()` 这个函数，这个时候 element 的队列中一定是有请求的，所以这个时候 `ssd_activate_elem()` 中的 `elem->metadata.reqs_waiting > 0` 一定是成立的，所以这个时候后台 GC 与普通的 GC 是没有太大区别的。所以说后台 GC 与普通 GC 最大的区别在于从 `ssd_media_access_request_element()` 这个函数进入到 `ssd_activate_elem()`。

我们假设这样的一种情况，当一个 element 上的请求全部完成了，恰恰这个时候 element 上没有多余的 page 可以用了。

如果有后台 GC，这个时候从 `ssd_media_access_request_element()` 进入到 `ssd_activate_elem()` 时，会进行 GC 的判断，由于没有空闲块了，那么就会进行 GC 操作，这个时候该 element 上已经没有请求了，所以这个时候 GC 操作对 SSD 的响应时间是没有影响的。

如果没有后台 GC，只是普通 GC 的话，由于该 element 的队列上已经没有请求了，所以这个不会进行 GC 的判断，当该 element 上的下一个请求到来时，会触发 GC 操作，下一个请求的响应时间自然提高了。
layout: post
title: disksim with ssdmodel 源码解析（八）：syssim_driver 系统级接口(一)
date: 2016-3-19 11:23:22
tags: 
	- SSD
	- Disksim
comments: true
copyright: true
---


disksim 提供了一个系统级的调用接口，我们调用这个接口可以不停的给 disksim 发送请求，当请求完成时接口会触发一个回调函数做相应的操作。下面重点来介绍下这个接口。

接口的实现是在 `src/syssim_driver.c` 这个文件中。调用方式是 `syssim <parameters file> <output file> <max. block number>`。

在 `syssim_driver.c` 的 main 函数中，首先是调用函数 `disksim_interface_initialize()` 实例化一个接口，然后循环1000次，产生随机的1000个请求通过函数 `disksim_interface_request_arrive()` 发送给 disksim，并通过函数 `disksim_interface_internal_event()` 处理相应请求的事件。最后调用函数 ` disksim_interface_shutdown()` 关闭接口。

<!--more-->

下面重点分析下在1000次循环中是如何通过接口发送请求的。

```C
for (i=0; i < 1000; i++) {
	r.start = now;     // 一个事件的开始时间，是上一个事件的结束时间
	r.flags = DISKSIM_READ;
	r.devno = 0;

	r.blkno = BLOCK2SECTOR*(DISKSIM_lrand48()%(nsectors/BLOCK2SECTOR));  // 随机生成 blkno
	r.bytecount = BLOCK;
	completed = 0;
	disksim_interface_request_arrive(disksim, now, &r);  //发送请求

	while(next_event >= 0) {   // 当前请求全部完成了才会发送下一个请求
		now = next_event;
		next_event = -1;
		disksim_interface_internal_event(disksim, now, 0);//处理未完成的事件
	}

	if (!completed) {
		fprintf(stderr,"%s: internal error. Last event not completed %d\n", argv[0], i);
		exit(1);
	}
  }
```

上面的1000个请求都是等上一请求全部完成时才会发送下一个请求。很显然这是不符合实际情况的，毕竟在这里只是给了一个示例，告诉大家怎么用这个接口，怎么把这个接口改成唯我所用，按照我们的要求去发送请求，后面会详细介绍。今天主要介绍接口的使用。


值得注意的两点是请求的 blkno 和 bytecount。不管使用 ssdsim 还是 hddsim, disksim 内部操作的块都是以512B为单位，所以请求的 blkno 的单位为 512B，对于 hdd 而言，大小可以任意，而对ssd而言其必须为4k的整数倍，即该值必须是8的整数倍。 bytecount 的单位是字节，对于 ssd 而言同样应该是 4k 的倍数，即该值应该为4096的整数倍。

最后介绍下 `syssim_report_completion()`， 这个函数是请求完成的回调函数，每当一个请求完成时都会调用这个函数，所以可以在这个函数里面进行一些请求完成时的业务逻辑操作，比如请求的响应时间的计算。


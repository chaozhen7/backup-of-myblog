layout: post
title: disksim with ssdmodel 源码解析（二十一）：warm up ssd
date: 2016-4-15 21:23:22
tags: 
	- SSD
	- Disksim
comments: true
copyright: true
---

一个新的 SSD 其性能总是很好的，随着不断的使用性能会变得越来越差，因为这个时候可能会产生更多的垃圾回收，也可能会产生一些坏块，这些都可能使 SSD 的性能变得很差。

我们在做实验的时候总是希望实验环境能模拟最真实的情况的，所以我们并不想每次在做实验的时候，trace 数据都是跑在一个全新的 SSD 上面。这个时候我们可以在真实实验之前给 SSD 做一次 warm up，也就是用一些 trace 先让 SSD 跑一会，然后再运行我们需要测试的 trace 文件。

<!--more-->

在 disksim 中提供了 warm up 的功能，在 disksim 的参数文件的 global 模块中有两个可选参数 `warm-up time` 和 `warm-up IOs`。官方的解释如下：
> warm-up time：This specifies the amount of simulated time after which the statistics will be reset.
Statistic warm-up IOs：This specifies the number of I/Os after which the statistics will be reset.

大概意思就是：warm-up time，这个参数表示在此时间之前的 trace 都是用来 warm up 的，在这之后的 trcace 才是用来统计的。warm-up IOs，表示前 warm-up IOs 条 trace 都是用来 warm up 的，之后的才是用来统计的。

假设我们要用某一个 tracefile 来 warm up 我们的 SSD，要怎么做呢，首先的看看这个 tracefile 有多少条 trace 数据。假设有 1000000 条，那么修改输入的参数文件，在 global 模板里添加：
```
Statistic warm-up IOs = 1000001,
```

不要问我为什么要比原来的多一个，现实就是这么奇妙。

然后可以在 `syssim_driver.c` 中自己写一个函数来实现。

```
//use tracefile to warm-up
static double warm_up(char filename[25],struct disksim_interface *disksim,int num){
	int i,devno,logical_block_number,size,isread;
	double time,warm_up_time;
	char line[201];
	warm_up_time = 0.0;
	for(i=0;i<num;i++){
		FILE  *warmup_file = fopen(filename,"r");
		if(warmup_file == NULL){
			fprintf(stderr, "we use %s to warm up ,but can't open it",filename);
			exit(1);
		}
		fgets(line,200,warmup_file);
		while(!feof(warmup_file)) {	  
			if(sscanf(line, "%lf %d %ld %d %d", &time, &devno, &logical_block_number,&size, &isread)!=5){
				fprintf(stderr, "Wrong number of arguments for I/O trace event type\n");
				fprintf(stderr, "line: %s", line);
			}
			struct disksim_request *  r  = malloc(sizeof(struct disksim_request)); 
			r->start = time + warm_up_time;
			r->flags = isread;
			r->devno = devno;
			/* NOTE: it is bad to use this internal disksim call from external... */
			r->blkno = logical_block_number;
			r->bytecount = size * 512;
			completed = 0;
			disksim_interface_request_arrive(disksim, time+warm_up_time, r);
			fgets(line,200,warmup_file);
		}
		fclose(warmup_file);
		warm_up_time  = warm_up_time + time;
	}
	return warm_up_time + 1000; // let the warm up trace completed
}
```

在你开始给 disksim 发送你自己的 trace 数据之前调用这个函数就行了。

`filename` 表示用哪个文件来做 warm up。

`num` 表示这个文件重复输入多少遍来做 warm up，这里重复的次数要跟参数文件中的 Statistic warm-up IOs 参数相对于，这个参数应该是等于文件中的 trace 条数乘以重复的次数 再加1（不要问我为啥要加1，反正就是这么奇妙）

最后的最后，这个函数返回的参数很重要，记得后面你给 disksim 发你需要测试的trace数据时，在每条 trace 的时间上加上这个函数返回的时间。


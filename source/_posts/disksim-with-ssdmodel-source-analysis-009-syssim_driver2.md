layout: post
title: disksim with ssdmodel 源码解析（九）：syssim_driver 系统级接口(二)
date: 2016-3-19 12:23:22
tags: 
	- SSD
	- Disksim
comments: true
copyright: true
---


前面介绍了 `syssim_driver` 这个 disksim 系统级接口。程序的作者留给我们的只是应用该接口的一个小例子。例子有很多的局限性，所以我们要通过一些修改才能唯我所用。

原程序最大的局限性就在于，每次发送请求都是等上一个请求全部完成了才发送下一个请求。很显然这是不切实际的，所以这一块是要我们修改的。还有一点就是要将原程序中随机生成的 trace 数据改成我们自定义的数据，可以改成从文件中去读取我们自己的 trace 。

原来程序的调用方式是 `syssim <parameters file> <output file> <max. block number>` ，`max. block number` 表示的是最大的 blkno 主要是在随机生成 trace 时用到，这里我们用不上这个参数，直接把这个参数换成我们的 trace 文件就好了。然后采用这样的方式调用该程序：`syssim <parameters file> <output file> <trace file>`

<!--more-->

修改比较简单直接看代码吧：

```c
int main(int argc, char *argv[])
{
	int i;
	int nsectors;
	struct stat buf;
	struct disksim_interface *disksim;

	if (argc != 4 ) {
		fprintf(stderr, "usage: %s <param file> <output file> <tracefile>\n",argv[0]);
		exit(1);
	}

	if (stat(argv[1], &buf) < 0)
		panic(argv[1]);

	disksim = disksim_interface_initialize(argv[1], argv[2],
			syssim_report_completion,
			syssim_schedule_callback,
			syssim_deschedule_callback,0,0);

	FILE *tracefile = fopen(argv[3], "r");
	double time;
	int devno;
	unsigned long logical_block_number;
	int size;
	int isread;
	double senttime=0.0;
	char line[201];
	fgets(line,200,tracefile);
	while(!feof(tracefile)) {   //开始发请求
		if(sscanf(line, "%lf %d %ld %d %d", &time, &devno, &logical_block_number,&size, &isread)!=5){
			fprintf(stderr, "Wrong number of arguments for I/O trace event type\n");
			fprintf(stderr, "line: %s", line);
		 }
		struct disksim_request *r = malloc(sizeof(struct disksim_request));  // 这里需要特别注意，每个request都需要不同的内存
		r->start = time;
		r->flags = isread;
		r->devno = devno;
		r->blkno = logical_block_number;
		r->bytecount = size * 512;  // 必须是一个块(512B)的整数倍
		disksim_interface_request_arrive(disksim, time, r);
		fgets(line,200,tracefile);
	}
	fclose(tracefile);
	while(next_event >= 0) {    //处理剩下的事件
		now = next_event;
		next_event = -1;
		disksim_interface_internal_event(disksim, now, 0);
	}

	disksim_interface_shutdown(disksim, now);

	print_statistics(&st, "response time");

	exit(0);
}

```

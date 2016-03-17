layout: post
title: disksim with ssdmodel 源码解析（四）：ssd_event_arrive 函数的介绍
date: 2016-3-13 20:20:22
tags: 
	- Disksim
	- SSD
comments: true
---


在 Disksim 中所有的工作都是由事件来完成，每个部件都有自己的事件，通过事件来驱动其他部件事件的状态的改变。

今天我们来看看 ssd.c 中的第一个函数 `ssd_event_arrive(curr)`，他是 ssdmodel 的入口函数。

在进入 `ssd_event_arrive` 函数后我们可以用 gdb 工具的 bt 命令来查看下函数的堆栈情况，看看是在什么函数里调用了 ssd_event_arrive 才进入到 ssdmodel 的。
 
disksim 会在两种情况下进入到 ssdmodel　中，第一种情况是在建立 disksim 时，也就是 `disksim_setup_disksim` 函数中；第二种中就是在 disksim 模拟磁盘操作时，也就是在 `disksim_run_simulation` 函数中

<!--more-->


**第一种情况**的函数调用嵌套： main-> disksim_setup_disksim ->  initialize ->  io_initialize -> iodriver_initialize -> get_device_maxoutstanding -> iodriver_send_event_down_path -> bus_deliver_event -> controller_event_arrive -> controller_passthru_event_arrive ->  bus_deliver_event -> device_event_arrive -> ssd_event_arrive .

在这种情况下进入 ssd_event_arrive 后，curr->type 为 `IO_QLEN_MAXCHECK`，如执行如下的代码：

```C
case IO_QLEN_MAXCHECK:
	/* Used only at initialization time to set up queue stuff */
	curr->tempint1 = -1;
	curr->tempint2 = ssd_get_maxoutstanding(curr->devno);
	curr->bcount = 0;
	break;
```

这部分的逻辑代码，只有通过 disksim_setup_disksim 进入到 ssdmodel 时才会被执行到，ssd_get_maxoutstanding(curr->devno) 其实返回的就是 getssd(curr->devno)->maxqlens。


**第二种情况**下进入到 ssdmodel 时，函数调用嵌套关系如下：

```python
ssd_event_arrive	# at ssd.c:924
device_event_arrive	# at disksim_device.c:388
bus_deliver_event	# at disksim_bus.c:522
controller_bus_delay_complete	# at disksim_controller.c:385
bus_event_arrive	# at disksim_bus.c:293
io_internal_event	# at disksim_iosim.c:268
disksim_simulate_event	# at disksim.c:794
disksim_run_simulation	# at disksim.c:1067
main	# at disksim_main.c:79
```


这种情况下进入 ssd_event_arrive 函数表明开始对SSD进行操作了。第一次进入的是 `IO_ACCESS_ARRIVE`，一个IO事件的到来，进入之后加上 一个IO请求调度开销 `currdisk->overhead` 之后重新加入事件队列中进行调度，状态转变成 `DEVICE_OVERHEAD_COMPLETE`。下一次调度之后就进入 `DEVICE_OVERHEAD_COMPLETE`，调用`ssd_request_arrive(curr)` 函数了。


`ssd_request_arrive(curr)` 函数就是进行 SSD 访问了，首先将此请求加入 SSD 请求队列，查看是否可以调度，如果可以则进行连接准备进行 IO。如果是读操作的话，就要读取 SSD，将数据读出来。如果是写操作则将数据传输进来，将时间状态转换成一个 IO 中断事件，准备进行数据传输（读操作将数据传输到SSD，写操作将数据传输到主机）。

接下来进入 `IO_INTERRUPT_COMPLETE` 中，执行 `ssd_interrupt_complete` 中的 `reconnect` 中执行数据传输；

接下来进入 `DEVICE_DATA_TRANSFER_COMPLETE`，完成数据传输，读操作将数据读到主机，写操作将数据写入 SSD；

接下来还是中断事件，进行善后操作，比如看看SSD等待队列中是否还是其他请求在等待，准备下一轮操作。

代码如下：
```C
void ssd_event_arrive (ioreq_event *curr)
{
	ssd_t *currdisk;

   	// fprintf (outputfile, "Entered ssd_event_arrive: time %f (simtime %f)\n", curr->time, simtime);
   	// fprintf (outputfile, " - devno %d, blkno %d, type %d, cause %d, read = %d\n", curr->devno, curr->blkno, curr->type, curr->cause, curr->flags & READ);
   
	//获取SSD结构体，包括了所有SSD的相关数据。
   	currdisk = getssd (curr->devno);
   
   	switch (curr->type) {

      		case IO_ACCESS_ARRIVE:
         		curr->time = simtime + currdisk->overhead;
         		curr->type = DEVICE_OVERHEAD_COMPLETE;
         		addtointq((event *) curr);
         		break;
		//设备请求，执行设备访问，读操作读SSD，写操作将数据传送进来
      		case DEVICE_OVERHEAD_COMPLETE:
         		ssd_request_arrive(curr);
        		 break;
		//设备访问完成事件
     	 	case DEVICE_ACCESS_COMPLETE:
         		ssd_access_complete (curr);
         		break;
		//数据传送事件
      		case DEVICE_DATA_TRANSFER_COMPLETE:
         		ssd_bustransfer_complete(curr);
         		break;
		//设备中断事件
      		case IO_INTERRUPT_COMPLETE:
         		ssd_interrupt_complete(curr);
        	 	break;

		case IO_QLEN_MAXCHECK:
         
         		curr->tempint1 = -1;
         		curr->tempint2 = ssd_get_maxoutstanding(curr->devno);
         		curr->bcount = 0;
         		break;

      		case SSD_CLEAN_GANG:
          		ssd_clean_gang_complete(curr);
          		break;

      		case SSD_CLEAN_ELEMENT:
          		ssd_clean_element_complete(curr);
          		break;

        	default:
         		fprintf(stderr, "Unrecognized event type at ssd_event_arrive\n");
         		exit(1);
   	}

   	// fprintf (outputfile, "Exiting ssd_event_arrive\n");
}
```

</br>
整理自：http://blog.sina.com.cn/s/blog_4b003c5501017v3p.html



layout: post
title: disksim with ssdmodel 源码解析（五）：程序流程介绍
date: 2016-3-16 13:23:22
tags: 
	- SSD
	- Disksim
comments: true
copyright: true
---

程序的主函数(main函数)在 `src/disksim_main.c` 文件里，代码很简洁，如下：

<!--more-->

```
int main (int argc, char **argv)
{
	int len;

#ifndef _WIN32
	setlinebuf(stdout);
	setlinebuf(stderr);
#endif

	if(argc == 2) {
		disksim_restore_from_checkpoint (argv[1]);
	} 
	else {
		disksim = calloc(1, sizeof(struct disksim));
		disksim_initialize_disksim_structure(disksim);
		disksim_setup_disksim (argc, argv);
	}
	disksim_run_simulation ();
	disksim_cleanup_and_printstats ();
	exit(0);
}
```

除了输入参数的检查可以大致分为如下几个步骤：

### **1. 给全局变量 `disksim` 分配内存** ###

这里 `disksim` 变量没有声明直接使用了，其实他是一个全局变量，在 `disksim.c` 的 120 行左右有其初始化的语句：`disksim_t *disksim = NULL`。

### **2. 初始化 disksim 结构体** ###

这里调用的是 `disksim_initialize_disksim_structure(disksim)` 这个函数，函数内容很简单，如下：
```c
int disksim_initialize_disksim_structure (struct disksim *disksim){
	
	disksim->closedthinktime = 0.0;
	// warmuptime 对应的宏定义为 (disksim->warmuptime)
	warmuptime = 0.0;	/* gets remapped to disksim->warmuptime */
	// simtime 对应的宏定义为  (disksim->simtime)        
	simtime = 0.0;	/* gets remapped to disksim->warmuptime */
	disksim->lastphystime = 0.0;
	disksim->checkpoint_interval = 0.0;

	return 0;
}
``` 

### **3. 建立 disksim ** ###

这里调用的是 `disksim_setup_disksim (argc, argv)` 这个函数，这个函数比较复杂，主要是对 disksim 做一些初始化操作。大致功能如下：设置对齐方式(大端对齐还是小端对齐)，设置输出文件，设置 trace 的格式，设置输入的 trace 数据的文件，判断是否启用 synthgen，设置重写的参数，根据配置文件设置参数，iosim_info 的初始化，最后就是为开始模拟磁盘做准备。后面会详细介绍这个函数的功能。

### **4. 开始模拟** ###

前面主要是做一些准备工作，这里开始模拟磁盘的操作，通过 `disksim_run_simulation ()` 这个函数来开始模拟，这个函数并没有传入任何参数，可见模拟的是全局的 disksim 。

### **5. 清理现场，保存结果** ###

到这一步，磁盘的模拟工作已经结束了，接下主要是释放程序占用的内存，关闭相应的文件，然后打印输出程序的运行结果。


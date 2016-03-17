layout: post
title: disksim with ssdmodel 源码解析（六）：disksim_setup_disksim
date: 2016-3-16 14:23:22
tags: 
	- SSD
	- Disksim
comments: true  
---

`disksim_setup_disksim()` 是在 main 函数中被调用的，主要用来设置 disksim 各种参数，为磁盘模拟做准备。函数的源码在 `disksim.c` 文件中。对于函数的功能主要有以下几个方面：


### **1. 设置对齐方式** ###

通过调用 `setEndian()` 这个函数来设置大端对齐还是小端对齐。

<!--more-->

### **2. 设置输出文件** ###

通过调用 `disksim_setup_outputfile` 这个函数来设置输出文件。首先是判断是否输出到控制台，如果是的话， `disksim->outputfile = stdout`，如果不是的话，则执行 `disksim->outputfile = fopen(filename,"w")` 和 `strcpy (disksim->outputfilename, filename)`。

### **3. 判断是否使用外部 trace 类型** ###

如果 trace 类型为 external 则 `disksim->external_control = 1`，否则调用 `iotrace_set_format()` 这个函数设置 trace 类型。iotrace_set_format() 首先会判断 disksim->iotrace_info 是否为 NULL，是的话会调用 iotrace_initialize_iotrace_info() 函数初始化 disksim->iotrace_info；然后紧接着设置  `disksim->traceendian = _LITTLE_ENDIAN` 和 `disksim->traceheader = TRUE`；最后根据输入的 trace 类型设置 `disksim->traceformat` 的值。

### **4. 设置 trace 文件** ###

调用 `disksim_setup_iotracefile()` 函数设置tarce文件，如果使用用户自定义的trace文件，`disksim->iotrace = 1`，`disksim->iotracefile = fopen(filename,"rb"))`；如果适用系统自己生成的trace，那么`disksim->iotracefile = NULL`。

### **5. 判断是否启用 synthgen** ###

根据输入参数判断是否启用 synthgen，如果启用的话，`disksim->synthgen = 1`，`disksim->iotrace = 0`，否则不操作。

### **6. 参数重写** ###

设置 `disksim->overrides` 和 `disksim->overrides_len` 的值来设置重写参数。

### **7. 导入参数** ###

通过 `disksim_loadparams()` 函数，导入参数设置 disksim。

### **8. 初始化 iosim_info** ###

如果输入参数文件里没有提供 iosim 块，那么这里需要调用 `iosim_initialize_iosim_info()` 函数初始化 iosim。

### **8. 初始化** ###

这里会掉用一个 `initialize()` 函数。主要功能是：
1. 根据输入参数设置 disksim->synthgen，为1或者为0，表示开启或者不开启 synthgen。
2. 调用 `iotrace_initialize_file()`，主要是针对 traceformat == HPL时的情况，其他情况该函数不做任何操作。
3. 将 `disksim->intq` 队列中的事件取出来添加到到 `disksim->extraq` 队列中。
4. 根据情况是否初始化 IO，即执行 `io_initialize()` 函数。
5. 如果使用 synthgen 初始化cpu，即调用 `pf_initialize()` 函数，否则初始化随机数种子，调用 `DISKSIM_srand48()` 函数
6. 初始化时间，`disksim->simtime = 0`。

### **9. 准备模拟** ###

为 disksim 的模拟操作做准备，调用 `prime_simulation()` 函数来完成。
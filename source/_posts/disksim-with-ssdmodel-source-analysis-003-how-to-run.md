layout: post
title: disksim with ssdmodel 源码解析（三）：初次使用介绍
date: 2016-3-12 14:20:22
tags: 
	- Disksim 
	- SSD
comments: true
---

disksim 是由卡内基梅隆大学开发的一个磁盘模拟器，其目的是：理解存储系统的性能，对新的存储结构进行评估。 包含许多存储元件的模型：`device drivers`, `buses`, `controllers`, `adapters`, `disk drives`。

ssdmodel 是微软基于 disksim-4.0 开发的一个插件，主要是针对 SSD 的。

前面已经介绍过了 [disksim-4.0 的安装](../../../../2015/09/09/install-disksim-4.0-with-ssdmodel-on-ubuntu/)，包括安装 ssdmodel 模块。

下面主要介绍下 disksim-4.0 的使用。

<!--more-->

在 `disksim-4.0/valid` 和 `disksim-4.0/ssdmodel/valid/` 下都有一个 `runvalid` 脚本，里面有运行示例，可以参考。

整个系统里面有两个文件包含可以运行的主函数： `Disksim.c` 和 `Syssim_driver.c`, 下面分别介绍。

## **Disksim.c** ##

disksim.c 运行至少需要输入五个参数，格式为： `disksim <parfile> <outfile> <tracetype> <tracefile> <synthgen> [par_override…]`

disksim 为可执行程序的名称，在安装目录的/src子目录里。

**parfile**

参数文件，包括仿真器各个参数的配置。欲知详情可以看之前的一篇文章: [disksim 中输入参数的介绍](../../../../2016/03/06/introduction-of-parameters-of-disksim-4.0-with-ssdmodel/)

**outfile**

输出文件，outfile 的项目内容可在 parfile 中设置，以去掉不需要的内容。

**tracetype**

确定输入trace的格式。比如：ASCII HPL HPL2 DEC VALIDATE RAW ATABUS IPEAK P OSTGRES EMCSYMM DEFAULT(ASCII)等等。

Disksim本身支持的trace类型是有限的，如果不支持实验使用的trace，则需要修改disksim源程序以添加新的trace类型，添加方法如下： (1)在 disksim_global.h 中预定义 trace 类型常量； (2)在 disksim_iotrace.c 文件的 iotrace_set_format 函数中添加相应的 if 判断语句； (3)在 disksim_iotrace.c 文件中仿照 iotrace_ascii_get_ioreq_event 函数添加 iotrace_spc_get_ioreq_event 函数处理相应的 trace 文件。如果trace文件中包含一定的头信息，则需要在 disksim_iotrace.c 文件中添加 iotrace_xxxx_initialize_file 函数，并在 iotrace_initialize_file 函数中调用。(具体可参考官方手册)

**tracefile**

输入的trace文件。 如果为 0 的话表示使用系统自动生成的trace数据。如果这个参数为 0 ，那么 synthgen 参数必须要大于0，表示开启合成负载。

trace 每一行包含5个参数值（用空格隔开）来描述一个磁盘请求:
- 请求到达时间（double 型，表示从模拟开始到请求发生的时间，必须按时间上升的顺序
- 设备号（int 型，指定设备号，如果有设备映射，被加到该值)
- 块号（int 型，表示请求的第一个设备地址，其值为逻辑设备可访问的单元）
- 请求大小（int 型）
- 请求标志（十六进制整数，有效值在disksim_global.h中定义）

**synthgen**

决定合成负载部分的模拟器是否打开（除0外的数表示开启）

当 synthgen 参数不为0时，那么 tracefile 参数就需要设置为0，synthgen的值就表示了用多少个 generator 来生成 trace,一个 generator 代表一个产生io请求的进程。每个 generator 的行为要在参数文件里面设置

模块可以配置为生成多钟合成的工作负载，与磁盘位置访问和请求到达时间相关。每一个合成器在简单系统模型里作为一个进程，在一定的“think time”后发出I/O请求，并等待完成。


对于特定的需求，实验时需要调整参数文件中 `disksim_synthgen` 配置或修改 `disksim_synthetic.c` 文件中的函数 `synthio_generatenextio` 以满足特定的请求特征（如访问的hot/cold特征，zipfan分布等），然后编译运行，获得实验结果。

Disksim提供了 uniform，normal，possion，expontenial,twovalue 等概率分布，可以灵活运用这些分布生成具有特定特征的请求，如符合zipfan分布的请求


**par_override** 

允许默认参数值或者parfile文件中的参数值替换为命令行指定的值。


## **Syssim_driver.c** ##

disksim 提供了一个接口（disksim_interface.c），可将 disksim 作为一个子系统，通过外部发请求获取实验结果。 Syssim_driver.c 就是利用这个接口去调用 disksim 来模拟磁盘的。

Syssim_driver.c 运行方式: `syssim <parameters file> <output file> <max. block number>`
例如： `syssim parv.seagate out 2676846`
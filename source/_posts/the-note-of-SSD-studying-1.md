layout: post
title: SSD 编程学习笔记（一）
date: 2016-3-29 22:23:22
tags: 
	- SSD
comments: true
copyright: true
---


看论文时整理的一些笔记，比较混乱，估计也只有自己能看懂吧。


### **影响SSD性能的一些因素** ###

**数据组织方式**：如果能提供一种有效的方式来组织 SSD chip 上的数据，不仅可以有助于负载均衡，对磨损平衡也能产生积极影响。

**并行性**：存储组件可以协调地组织在一起实现操作上的并行性。

**写操作的顺序**：解决 SSD 上小的随机的写操作是一个很棘手的问题。

**负载管理**：SSD 的性能是受负载影响的。比如，有些设计对连续的负载有很好的性能，但是对非连续的负载就不能体现出很好的性能，反之亦然。

<!--more-->

首先介绍下普通SSD的层次结构，为了增加存储密度，通常会把几个 Flash chip 集成到一个模块上(称为 flash package)，一个 flash package 上的chip共享数据线，控制信号线是分离的。一个 chip 包含很多 die，一个 die 包含很多 plane ，一个 plane 又包含很多 block，一个 block 包含很多 page。

不同厂家的SSD的性能差别较大，所以以 Samsung’s K9XXG08UXM series NAND-flash 为例来介绍。

### **Samsung’s K9XXG08UXM series NAND-flash 介绍** ###

![Samsung 4GB flash internals](/img/articles/ssd/Samsung SSD.jpg)

A flash package 由两个2G的die组成，共享一个8位的串行I/O总线和一些公共的控制信号。两个 die 有独立的 chip enable 和 ready/busy 信号。所以一个 die 在接受命令和数据的时候另外一个 die 可以执行其他的操作。这个 package 同样支持两个 die 之间的交叉操作。


每个 die 有四个 plane，每个 plane 有 2048 个 block。每个 block 有 64个page，每个 page 大小为 4KB。每个 page 上除了数据外还有 128字节的元数据(metadata)，保存一些属性和差错检验等信息。


读操作以 page 位单位，一个读操作需要 25us 把数据从闪存读到数据寄存器，然后再把数据转发到数据总线。串行线每传输一个字节需要 25ns(传输一个 page 大概需要 100us)。

每一个擦除需要 1.5ms(以block为单位)，比读和写需要更多的时间。

每一个写操作也是以 page 为单位，把数据转发到寄存器上需要 100us，然后把它写到闪存单元中需要 200us。

block 中的 page 需要从低地址到高低地址顺序来写。


### **并行和交叉** ###

这个SSD同时提供了一个 copyback 功能，不需要通过串行数据线就可以从一个页面传输数据到另外一个页面。

接受命令和发送数据的串行接口是 SSD 性能的一个主要瓶颈。对于上述的 SSD，从 chip 上的寄存器发送一个 4KB 大小的页到 chip 外的控制器上需要100us。把数据从NAND存储单元移动到寄存器还需要 25us。如果这两个操作是串行的话，那么这个闪存一秒钟只能进行8000个页的读操作(32MB/s)，如果是以 die 交叉操作的话，那么单个 die 的读带宽可以提高到每秒钟10000个页的读操作(40MB/s)

写操作和读操作一样，数据的串行传输也需要 100us。但是把数据写到存储单元上还需要200us。如果没有交叉操作，单个部分的写带宽是每秒3330个页的写(13MB/s)。


当操作延迟比串行存取延迟大很多的时候，交叉操作可以很好提高速度。比如一个耗时的擦除指令在某些情况下可以和其他的指令并行。如果我们在两个package之间完全交叉的去复制页面的话，那么在忽略写数据耗时(200us)的情况下，每个页面的平均复制时间可以缩短到100us。

如图所示，在四个 source plane 和 destination plane 之间可以迅速的复制页，并不需要在同一个 plane-pair 上操作，它们可以自动的利用链接到他们die上的串行总线引脚，在每一个时间间隔(100us)都可以完成一次写操作。

![Interleaved page copying](/img/articles/ssd/ssd-page-copying.jpg)

即使闪存的架构支持交叉，在现实中还是会遇到很多限制，比如同一个 plane 上的操作不能交叉执行。前面提到的三星的 SSD 提供了一个内部的 copy-back 操作，允许数据从一个 block 复制到 chip 上的另外一个 block 上而不需要经过串行的引脚，当然只能在同一个  plane 上复制。同样的这两个操作是可以在不同的 plane 上交叉的。这种操作的性能提升和上图中描述的完全交叉是一样的，但是这种操作没有独占串行引脚。


### **内部逻辑结构** ###

下图是  SSD的一个结构图。每个 SSD 都需要一个主机接口来提供到不同物理主机的接口连接(USB,FiberChannel,PCI Express,SATA)，还需要一个逻辑磁盘仿真器(比如FTL)来使SSD能够模拟硬盘驱动。在整体上主机接口的带宽是设备影响的一个重要影响因素，所以它必须要和闪存的带宽相匹配。

![SSD Logic Components](/img/articles/ssd/ssd-logic-component.jpg)

内部的缓冲管理按照请求原始的数据路径保存着正在处理的请求。

多路转发器(Flash Demux/Mux)主要功能是发出指令，还有就是利用连接到flash package 上的串行接口发送数据。

处理器不仅需要管理请求流，还要提供逻辑的磁盘地址到物理的闪存地址的转换。

处理器，缓冲管理器和多路转发器通常是用独立的器件组成，比如 ASIC 或 FPGA，这些逻辑器件间的数据传输是非常快的。

如果大部分的逻辑块地址被静态分配，那么就只有很少的空间留给负载均衡。如果一个连续范围内的逻辑块地址映射到相同的物理块，那么顺序存储大多会受到影响。如果逻辑页比较小，那么从候选擦除页中选出有效页需要更多时间。如果逻辑页大小和block大小一致，那么擦除操作会很方便，但是所有比逻辑页小的写操作会导致 read-modify-write 操作。


### **gang 中 package 的组织方式** ###

组织一个gang中的package有两种方式：

第一种设计：package 连接到串行总线，根据指令来动态的选择package。如下图所示，

![SSD Logic Components](/img/articles/ssd/share-bus-gang.png)

数据总线和控制总线是共享的，每个 package 都有一个 enable line 可以根据命令来选择目标package。

这种设计可以不需要额外的引脚就可以增加性能，但是不会提高带宽。

第二种设计：每个 package 都独立的数据总线连接到控制器，但是控制引脚都是连接到一个广播总线上。如下图所示，

![SSD Logic Components](/img/articles/ssd/share-control-gang.png)

由于控制引脚是共享的，数据总线是独立的连接到每个 package 上，那么横跨多个 package 的操作是可以并行执行的。


第二种设计可以去掉 enable line，但是这样的话所有的操作都必须要应用到整个 gang 上，没有 package 是空闲的。


在一个 element 上进行长时间的操作(比如擦除操作）的同时可以在另一个element上进行其他的读写操作。这种方式唯一的限制就是对共享的数据总线和指令总线的竞争。

另一种形式的并行就是 plane 内部的 copy-back，可以被用来实现后台的垃圾回收操作。



## 参考文献 ##

整理自：*Agrawal N, Prabhakaran V, Wobber T, et al. Design Tradeoffs for SSD Performance[C]//USENIX Annual Technical Conference. 2008: 57-70.*
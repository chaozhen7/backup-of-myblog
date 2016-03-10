layout: post
title: 为 SSD 编程（2）：SSD的架构和基准
date: 2016-2-25 15:20:22
tags: 
	- SSD
comments: true
toc: true
---

本第二部分包含了“为SSD编程”的6个内容，包括1、2两节，你可以参考[目录](../../../../2016/02/25/coding-for-ssds-part-1-introduction-and-table-of-contents/)。这是我在阅读了各种关于SSD的文档之后，为了分享我所学到的东西写的一系列文章。如果你没有时间慢慢看，你可以直接跳转到[第六部分](../../../../2016/02/25/coding-for-ssds-part-6-a-summary-what-every-programmer-should-know-about-solid-state-drives/)，这部分总结了所有其他部分的内容。

在本部分，我将解释NAND闪存的基本知识、闪存单元类型、和基本SSD内部架构。同样包括了SSD基准和如何解释这些基准。

<!--more-->

![](http://codecapsule.com/wp-content/uploads/2014/02/ssd-presentation-02.jpg)

## **1.SSD的架构** ##

### **1.1 NAND闪存单元** ###

固态硬盘(SSD)是基于闪存的数据存储设备。每个数据位保存在由浮栅晶体管制成的闪存单元里。SSD整个都是由电子组件制成的，没有像硬盘那样的移动或者机械的部分。

在浮栅晶体管中，使用电压来实现每个位的读写和擦除。写晶体管有两个方法：NOR闪存和NAND闪存。我不会更加深入的讨论NOR和NAND闪存的不同。本文将只包含被大多数制造商采用的NAND闪存。更多关于NOR和 NAND的不同点的信息，你可以参考Lee Hutchinson写的这篇文章[31]

NAND闪存模块的一个重要特征是，他们的闪存单元是损耗性的，因此它们有一个寿命。实际上，晶体管是通过保存电子来实现保存比特信息的。在每个P/E循环（Program/Erase，“Program”在这表示写）中电子可能被晶体管误捕，一段是时间以后，大量电子被捕获会使得闪存单元不可用。

**有限的寿命**

每个单元有一个最大的P/E 循环数量，当闪存单元被认为有缺陷后，NAND闪存被损耗而拥有一个有限的寿命。不同类型的NAND闪存有不同的寿命[31]。

最近的研究表示，通过给NAND一个相当高的温度，被捕获的电子可以被清除[14, 51]。尽管这仍然还只是研究，并且还没有确定到底哪一天能够将这个研究应用的消费市场，但这确实可以极大地增加SSD的寿命。

目前业界中的闪存单元类型有：

- 单层单元（SLC），这种的晶体管只能存储一个比特但寿命很长。
- 多层单元（MLC），这种的晶体管可以存储2个比特，但是会导致增加延迟时间和相对于SLC减少寿命。
- 三层单元（TLC），这种的晶体管可以保存3个比特，但是会有更高的延迟时间和更短的寿命。


**闪存单元类型**

固态硬盘（SSD）是基于闪存的数据存储设备。比特存储在闪存单元中，有三种闪存单元类型：每个单元1比特（单层单元，SLC），每个单元2比特（多层单元，MLC），和每单元3比特（三层单元，TLC）。

下方的表1中是每种NAND类型的详细信息。为了比较，我添加了硬盘、内存、和L1/L2缓存的平均延迟。

	
|   |SLC  |  MLC |  TLC | HDD | RAM | L1 cache | L2 cache | 
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
|P/E循环 | 100k  | 10k | 5k | * | * | * | * | 
|每单元比特数量 | 1 | 2 | 3 | * | * | * | * | 
|寻址用时(μs) | * | * | * | 9000 | * | * | * | 
|读取用时(μs) | 25 | 50 | 100 | 2000-7000 | 0.04-0.1 | 0.001 | 0.004 | 
|写入用时(μs) | 250 | 900 | 1500 | 2000-7000 | 0.04-0.1 | 0.001 | 0.004 | 
|擦除用时(μs) | 1500 | 3000|  5000 | * | * | * | * |

表1：NAND闪存不同类型的特征和用时与其他记忆组件比较

备注:  * 该种计量不适用此种存储设备 
引用:
- P/E 循环 [20]  
- SLC/MLC用时 [1]  
- TLC用时 [23] 
- 硬盘用时 [18, 19, 25]  
- 内存用时 [30, 52] 
-  L1 和L2 缓存用时 [52] 

在相同数量的晶体管中的比特数更多可以降低生产成本。与基于MLC的SSD相比，基于SLC的SSD更可靠，并具有更长的寿命，但是有更高的生产成本。因此一般的大众SSD是基于MLC或者TLC的，只有专业的SSD是基于SLC的。因此往往会基于硬盘的目标工作负载和可能的数据更新频率，选择正确的存储类型。对于较高的更新工作负载，SLC是最后的选择，而高读取低写入的工作负载（例如视频存储和直播），TLC将极其适合。另外，TLC硬盘基于实际工作负载的基准检测显示出在实际中不必考虑基于TLC的SSD寿命。

**NAND闪存的页和块**

闪存的模块组织在被称为块的格子中，而块则组织成平面。块中可以读写的最小单元称为页。页不能独立擦除，只能整块擦除。NAND闪存的页大小可能是不一样的，大多数硬盘的页大小是2 KB, 4 KB, 8 KB 或 16 KB。大多数SSD的块有128或256页，这即表示块的大小也可能是256KB和4MB之间不同的值。例如Samsung SSD 840 EVO的块大小是2048KB，而每个块有256个8KB的页。页和块访问的细节在3.1节中

### **1.2 SSD的组织** ###

下方的图1是SSD硬盘及其组件的示例。我只是重复了数篇论文[2, 3, 6]中已有的基本示意图。

![](http://codecapsule.com/wp-content/uploads/2014/02/ssd-architecture.jpg)

图1：固态硬盘的架构

来自用户的命令是通过主机接口交换的。在我写这篇文章的时候，最新发布的SSD有两种最普遍的接口：SATA和PCIe。SSD控制器中的处理器接收这些命令并将它们传递给闪存控制器。SSD同样内嵌有RAM存储器，通常是作为缓存和存储映射信息使用。章节4包含了关于映射策略的细节信息。NAND闪存芯片通过多个通道组织在一起，和这些通道有关的信息在章节6中。

下方的图2和图3是从StorageReview.com [26, 27] 复制过来的，展示出真的SSD是长的什么样子的。
- 1个SATA3.0接口
- 1个SSD控制器（Samsung MDX S4LN021X01-8030）
- 1个RAM模块（256 MB DDR2 Samsung K4P4G324EB-FGC2）
- 8个MLC NAND闪存模块，每个模块有64G的存储空间（Samsung K9PHGY8U7A-CCK0）

![](http://codecapsule.com/wp-content/uploads/2014/01/samsungssd840pro-01.jpg)

![](http://codecapsule.com/wp-content/uploads/2014/01/samsungssd840pro-02.jpg)

图2：三星固态硬盘840 Pro（512 GB）— 感谢StorageReview.com[26] 的图片


图3是一个美光P420m 企业级PCIe固态硬盘，2013年末发布。要组件有：
- PCIe 2.0接口 x8
- 1个SSD控制器
- 1个RAM模块（DRAM DDR3）
- 32通道上的64个MLC NAND闪存模块，每个模块有32GB的存储空间（Micron 31C12NQ314?25nm）

总存储空间为2048GB，但在应用over-provisioning技术后只有1.4TB可用


![](http://codecapsule.com/wp-content/uploads/2014/01/micron-p420m-02.jpg)

图3：美光P420m企业级PCIe固态硬盘（1.4TB）— 感谢StorageReview.com[27] 的图片


### **1.3生产过程** ###

很多SSD的生产商使用表面贴装技术（SMT，电子组件直接放置在PCB板上的一种生产方法）来生产SSD。SMT生产线由一系列机器组成，每个机器上下衔接，并有各自生产过程中的任务，例如安放组件或者融化焊锡。整个生产过程中同样贯穿了多重质量检测。Steve Burke的两篇参观金士顿在加利福利亚芳泉谷市的生产工厂的文章[67, 68]和Cameron Wilmot的一篇关于台湾金士顿组装工厂的文章[69]中，有SMT生产线的照片和视频。

另外还有两个有趣的视频，一个是关于美光Crucial SSD[70]的，而另一个是关于金士顿[71]。后边一个视频是Steve Burke文章的一部分，我同样在下方引用了。金士顿的Mark Tekunoff领读者参观了他们SMT生产线。有意思的是，视频中的每个人都穿着一套萌萌的抗静电服，看起来很有意思。

## **2.基准和性能度量** ##

### **2.1基本基准** ###

下边的表2展示的是不同的固态硬盘在顺序和随机工作负载下的读写速度。为了便于比较，这里包含了2008年到2013年发布的SSD、一个硬盘盒、和一个内存芯片

|  | Samsung 64 GB | Intel X25-M | Samsung 840 EVO | Micron P420m | HDD | RAM | 
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
|品牌\\型号 | Samsung</br>(MCCDE64</br>G5MPP-OVA)|Intel X25-M(SSDSA2M</br>H080G1GC) | Samsung</br>(SSD 840 EVO mSATA) | Micron P420m | Western</br> Digital Black 7200 rpm | Corsair</br> Vengeance DDR3 |
|存储单元类型 | MLC | MLC | TLC | MLC | * | * |
|上市年份 | 2008 | 2008 | 2013 | 2013 | 2013 | 2012|
|接口 | SATA 2.0 | SATA 2.0 | SATA 3.0 | PCIe 2.0 | SATA 3.0 | * |
|总容量 | 64 GB | 80 GB | 1 TB | 1.4 TB | 4 TB | 4 x 4 GB|
|每块的页数 | 128 | 128 | 256 | 512 | * | * |
|页大小 | 4 KB | 4 KB | 8 KB | 16 KB | * | * |
|块大小 | 512 KB | 512 KB | 2048 KB | 8196 KB | * | * |
|顺序读取 (MB/s) | 100 | 254 | 540 | 3300 | 185 | 7233|
|顺序写入 (MB/s) | 92 | 78 | 520 | 630 | 185 | 5872|
|4KB 随机读取(MB/s) | 17 | 23.6 | 383 | 2292 | 0.54 | 5319 ** |
|4KB 随机写入 (MB/s) | 5.5 | 11.2 | 352 | 390 | 0.85 | 5729 ** |
|4KB 随机读取(KIOPS) | 4 | 6 | 98 | 587 | 0.14 | 105 |
|4KB 随机写入 (KIOPS) | 1.5 | 2.8 | 90 | 100 | 0.22 | 102 |

注意： * 存储设备不支持该指标； ** 以2MB的块测量,而非 4 KB
指标： MB/s: MB每秒； KIOPS: 千操作每秒,每秒1000输入/输出操作
引用：
- Samsung 64 GB[21]
- Intel X25-M[2, 28]
- Samsung SSD 840 EVO[22]
- Micron P420M[27]
- Western Digital Black 4 TB[25]
- Corsair Vengeance DDR3 RAM[30]

表2: 固态硬盘与其他存储设备比较，特征和读写速度

影响性能的一个重要因素是接口。最新发布的SSD最常使用的接口是SATA3.0和PCI Express 3.0。使用SATA3.0接口时，数据传输速度可以达到6 Gbit/s，而在实际上大概能够达到550MB/s。而使用PCIe 3.0可以达到每条8 GT/s，而实际上能达到大概1 GB/s（GT/s是指G次传输（Gigatransfers）每秒）。使用PCIe 3.0接口的SSD都会使用不止一条通道。使用4条通道的话（译注：PCIe 3.0 x4），PCIe 3.0可以提供最大4 GB/s,的带宽，相当于SATA3.0的四倍一些企业级的SSD同样提供串行SCSI接口（SAS），最新版本的SAS可以提供最高12 GBit/s的速度，但是现在SAS的市场占有量很小。

大部分近期的的SSD的内部速度可以满足550 MB/s的SATA3.0限制，因此接口是其速度瓶颈。使用PCI Express 3.0或者SAS的SSD可以提供巨大的性能提升。

**PCI Express 和SAS 比 SATA要快**

生产商提供的两个主要接口是SATA3.0(550MB/s)和PCI Express 3.0 (每通道1 GB/s, 使用多个通道)。串行SCSI（SAS）同样应用在企业级SSD上。最新版本的接口定义中PCI Express 和SAS 比 SATA要快，但是同样要更贵。

### **2.2 预处理** ###

> 如果你折磨数据足够久，它会招的 —— Ronald Coase


SSD生产商提供的数据资料充斥着令人惊讶的性能值。确实，通过各种乱七八糟的方法对数据处理足够长的时间，生产商似乎总能找到方法在商业传单上显摆那些闪亮的数字。这些数字是否真的有意义，或者能否真的反映产品系统的性能则是另外的问题。Marc Bevand在他关于常见SSD性能基准缺陷的文章中[66]，提到了一些例子。例如常见的报道随机写负载的IOPS（每秒读写操作次数）而不提所跨的LBA（逻辑区块地址）的范围，很多IOPS的数据只是基于队列深度为1，而非整个硬盘最大值而测试的。同样也有很多基准性能测试工具误用的例子。

正确评估SSD的性能并非易事。硬件评测博客上的很多文章都是在硬盘上随机写十分钟，便声称硬盘可以进行测试，并且测试结果是可信的。然而SSD性能只会在足够长时间的随机写工作负载下才会有性能降低，而所需的时间基于SSD的总大小会花费30分钟到3小时不等。这即是更多认真的基准性能测试开始于相当长时间的随机写负载（称为“预处理”）的原因[50]。下方的图7来自StorageReview.com上的一篇文章[26]，显示出在多个SSD上预处理的效果。可以看见在30分钟左右出现了明显的性能下降，所有硬盘都出现读写速度下降和延迟上升。之后的四个小时中，硬盘性能缓慢降低到一个最小的常量值。

![](http://codecapsule.com/wp-content/uploads/2014/01/writes-preconditioning.jpg)

图7：多个SSD上预处理的效果 — 感谢来自StorageReview.com[26]的图片

5.2节解释了图7中实际上发生的事情，随机写入的量太大并以这种持续的方式进行使得垃圾回收进程不能维持在后台。因为必须在写命令到达时擦除块，因此垃圾回收进程必须和来自主机的工作在前台的操作竞争。使用预处理的人声称基准测试可以代表硬盘在最坏的情况下的表现。这种方法在所有工作负载下是否都是好模型还是值得商榷。

为了比较不同制造商的各种产品，找到可以比较的共同点是必要的，而最坏的情况是一个有效的共同点。然而选择在最糟糕的工作负载下表现最好的硬盘并不能保证其在生产环境下所有的工作负载下都表现的最好。实际上大部分的生产环境下，SSD硬盘只会在唯一的一个系统下工作。因其内部特征，这个系统有一个特定的工作负载。因此比较不同硬盘的更好更精确的方法是在这些硬盘上运行完全相同的工作负载，然后比较他们表现的性能。 这就是为何，即使使用持续的随机写工作负载的预处理可以公平的比较不同SSD，但还是有一点需要注意，如果可以的话，运行一个内部的基于目标工作负载的基准测试。

内部基准测试同样可以通过避免使用“最好的”SSD来避免过度调配资源，譬如当一个比较便宜的SSD型号已经足够并且能够省下一大笔钱的时候。

**基准测试很难**

测试者是人，因此并不是所有的基准测试都是没有错的。在使用生产商或者第三方的基准测试的时候请小心，并且在相信这些数据之前参考多个消息源，孤证不立。如果可以的话，使用你系统的特定工作负载来进行你自己的内部基准测试。

### **2.3工作负载和指标** ###

性能基准都有相同的指标，并使用相同的度量。在本节中，对于如何解释这些指标，我希望能够给出一些见解。

通常使用的参数如下：

- 工作负载类型：可以是基于用户控制数据的指定性能基准，或者只是顺序或者随机访问的性能基准（例：仅随机写）
- 读写百分比（例：30%读70%写）
- 队列长度：在硬盘上运行命令的并发执行线程的数量
- 访问的数据块大小（4 KB 8 KB等）

基准测试的结果可能使用不同的度量指标。常用的如下：

- 吞吐量：数据传输的速度，通常单位是KB/s或MB/s，表示千字节每秒和百万字节每秒。这个指标常用在顺序读写基准测试中。
- IOPS:每秒读写操作的数量，每个操作都是相同大小的数据块（通常是4KB/S）。这个指标通常用在随机读写基准测试中。[17]
- 延迟：在发送完命令后设备的反应时间，通常是μs或ms，表示微秒或者毫秒。


虽然吞吐量这个指标很容易理解和接受，但IOPS却比较难以领会。例如，如果一个硬盘在随机写上的表现是在4KB的数据块上是1000 IOPS，这即表示吞吐量是1000 x 4096 = 4 MB/s.。因此，IOPS高只有在数据块足够大的时候才可以被解释成吞吐量高，而IOPS高但平均数据块小的话只能代表一个低吞吐量。

为了阐明这个观点，不妨想象我们有一个登陆服务器，每分钟要对数千个不同的文件执行微量的更新，表现出10k IOPS的性能。因为这些更新分布在如此多的文件里，吞吐量能够接近20 MB/s，然而在同一个服务器中仅对同一个文件进行顺序写入能够将吞吐量提高到200 MB/s，这可是10倍的提升。例子中的数字是我编的，不过这些数据很接近我接触到的生产环境。

另一个需要掌握的概念是，高吞吐量并不足以表示这是一个快的系统。实际上，如果延迟很高，不管吞吐量有多么好，整个系统还是会慢。让我们拿一个假象的单线程进程为例，这个进程需要连接25个数据库，每个数据库都有20ms的延迟。因为连接的延迟是累积的，获取25个连接需要5 x 20 ms = 500 ms。因此，即便运行数据库查询的机器有很快的网卡，就当5 GBits/s的带宽吧，但这个脚本仍然因为延迟而会很慢。

本节的重点在于，着眼于全部的指标是很重要的，这些指标会显示出系统的不同特征，并且可以在瓶颈出现时识别出来。当在看SSD的基准测试并确定所选择的型号时，通常有一个很好的经验就是，别忘了这些待选的SSD中，哪个指标对于系统是最关键的。

对于这个主题有一个很有意思的扩展阅读：Jeremiah Peschka 写的一篇文章“IOPS是骗局”[46]。


## **引用** ##

[1]<a href="http://www.cse.ohio-state.edu/hpcs/WWW/HTML/publications/papers/TR-09-2.pdf" target="_blank">Understanding Intrinsic Characteristics and System Implications of Flash Memory based Solid State Drives, Chen et al., 2009</a>
[2]<a href="http://csl.skku.edu/papers/CS-TR-2010-329.pdf" target="_blank">Parameter-Aware I/O Management for Solid State Disks (SSDs), Kim et al., 2012</a>
[3]<a href="http://bit.csc.lsu.edu/~fchen/paper/papers/hpca11.pdf" target="_blank">Essential roles of exploiting internal parallelism of flash memory based solid state drives in high-speed data processing, Chen et al, 2011</a>
[4]<a href="http://ieeexplore.ieee.org/xpls/abs_all.jsp?arnumber=6165265" target="_blank">Exploring and Exploiting the Multilevel Parallelism Inside SSDs for Improved Performance and Endurance, Hu et al., 2013</a>
[5]<a href="http://research.microsoft.com/pubs/63596/usenix-08-ssd.pdf" target="_blank">Design Tradeoffs for SSD Performance, Agrawal et al., 2008</a>
[6]<a href="http://instartlogic.com/resources/research_papers/aanand-tunable_efficient_ssd_indexes.pdf" target="_blank">Design Patterns for Tunable and Efficient SSD-based Indexes, Anand et al., 2012</a>
[7]<a href="https://www.usenix.org/legacy/events/fast08/tech/full_papers/kim/kim.pdf" target="_blank">BPLRU: A Buffer Management Scheme for Improving Random Writes in Flash Storage, Kim et al., 2008</a>
[8]<a href="https://www.usenix.org/legacy/event/fast12/tech/full_papers/Min.pdf" target="_blank">SFS: Random Write Considered Harmful in Solid State Drives, Min et al., 2012</a>
[9]<a href="http://idke.ruc.edu.cn/people/dazhou/Papers/AsurveyFlash-JSA.pdf" target="_blank">A Survey of Flash Translation Layer, Chung et al., 2009</a>
[10]<a href="http://idke.ruc.edu.cn/people/dazhou/Papers/a38-park.pdf" target="_blank">A Reconfigurable FTL (Flash Translation Layer) Architecture for NAND Flash-Based Applications, Park et al., 2008</a>
[11]<a href="https://www.usenix.org/legacy/event/fast11/tech/full_papers/Wei.pdf" target="_blank">Reliably Erasing Data From Flash-Based Solid State Drives, Wei et al., 2011</a>
[12]<a href="http://en.wikipedia.org/wiki/Solid-state_drive" target="_blank">http://en.wikipedia.org/wiki/Solid-state_drive</a>
[13]<a href="http://en.wikipedia.org/wiki/Write_amplification" target="_blank">http://en.wikipedia.org/wiki/Write_amplification</a>
[14]<a href="http://en.wikipedia.org/wiki/Flash_memory" target="_blank">http://en.wikipedia.org/wiki/Flash_memory</a>
[15]<a href="http://en.wikipedia.org/wiki/Serial_ATA" target="_blank">http://en.wikipedia.org/wiki/Serial_ATA</a>
[16]http://en.wikipedia.org/wiki/Trim_(computing)
[17]<a href="http://en.wikipedia.org/wiki/IOPS" target="_blank">http://en.wikipedia.org/wiki/IOPS</a>
[18]<a href="http://en.wikipedia.org/wiki/Hard_disk_drive" target="_blank">http://en.wikipedia.org/wiki/Hard_disk_drive</a>
[19]<a href="http://en.wikipedia.org/wiki/Hard_disk_drive_performance_characteristics" target="_blank">http://en.wikipedia.org/wiki/Hard_disk_drive_performance_characteristics</a>
[20]<a href="http://centon.com/flash-products/chiptype" target="_blank">http://centon.com/flash-products/chiptype</a>
[21]<a href="http://www.thessdreview.com/our-reviews/samsung-64gb-mlc-ssd/" target="_blank">http://www.thessdreview.com/our-reviews/samsung-64gb-mlc-ssd/</a>
[22]<a href="http://www.anandtech.com/show/7594/samsung-ssd-840-evo-msata-120gb-250gb-500gb-1tb-review" target="_blank">http://www.anandtech.com/show/7594/samsung-ssd-840-evo-msata-120gb-250gb-500gb-1tb-review</a>
[23]<a href="http://www.anandtech.com/show/6337/samsung-ssd-840-250gb-review/2" target="_blank">http://www.anandtech.com/show/6337/samsung-ssd-840-250gb-review/2</a>
[24]<a href="http://www.storagereview.com/ssd_vs_hdd" target="_blank">http://www.storagereview.com/ssd_vs_hdd</a>
[25]<a href="http://www.storagereview.com/wd_black_4tb_desktop_hard_drive_review_wd4003fzex" target="_blank">http://www.storagereview.com/wd_black_4tb_desktop_hard_drive_review_wd4003fzex</a>
[26]<a href="http://www.storagereview.com/samsung_ssd_840_pro_review" target="_blank">http://www.storagereview.com/samsung_ssd_840_pro_review</a>
[27]<a href="http://www.storagereview.com/micron_p420m_enterprise_pcie_ssd_review" target="_blank">http://www.storagereview.com/micron_p420m_enterprise_pcie_ssd_review</a>
[28]<a href="http://www.storagereview.com/intel_x25-m_ssd_review" target="_blank">http://www.storagereview.com/intel_x25-m_ssd_review</a>
[29]<a href="http://www.storagereview.com/seagate_momentus_xt_750gb_review" target="_blank">http://www.storagereview.com/seagate_momentus_xt_750gb_review</a>
[30]<a href="http://www.storagereview.com/corsair_vengeance_ddr3_ram_disk_review" target="_blank">http://www.storagereview.com/corsair_vengeance_ddr3_ram_disk_review</a>
[31]<a href="http://arstechnica.com/information-technology/2012/06/inside-the-ssd-revolution-how-solid-state-disks-really-work/" target="_blank">http://arstechnica.com/information-technology/2012/06/inside-the-ssd-revolution-how-solid-state-disks-really-work/</a>
[32]<a href="http://www.anandtech.com/show/2738" target="_blank">http://www.anandtech.com/show/2738</a>
[33]<a href="http://www.anandtech.com/show/2829" target="_blank">http://www.anandtech.com/show/2829</a>
[34]<a href="http://www.anandtech.com/show/6489" target="_blank">http://www.anandtech.com/show/6489</a>
[35]<a href="http://lwn.net/Articles/353411/" target="_blank">http://lwn.net/Articles/353411/</a>
[36]<a href="http://us.hardware.info/reviews/4178/10/hardwareinfo-tests-lifespan-of-samsung-ssd-840-250gb-tlc-ssd-updated-with-final-conclusion-final-update-20-6-2013" target="_blank">http://us.hardware.info/reviews/4178/10/hardwareinfo-tests-lifespan-of-samsung-ssd-840-250gb-tlc-ssd-updated-with-final-conclusion-final-update-20-6-2013</a>
[37]<a href="http://www.anandtech.com/show/6489/playing-with-op" target="_blank">http://www.anandtech.com/show/6489/playing-with-op</a>
[38]<a href="http://www.ssdperformanceblog.com/2011/06/intel-320-ssd-random-write-performance/" target="_blank">http://www.ssdperformanceblog.com/2011/06/intel-320-ssd-random-write-performance/</a>
[39]<a href="http://en.wikipedia.org/wiki/Native_Command_Queuing" target="_blank">http://en.wikipedia.org/wiki/Native_Command_Queuing</a>
[40]<a href="http://superuser.com/questions/228657/which-linux-filesystem-works-best-with-ssd/" target="_blank">http://superuser.com/questions/228657/which-linux-filesystem-works-best-with-ssd/</a>
[41]<a href="http://blog.superuser.com/2011/05/10/maximizing-the-lifetime-of-your-ssd/" target="_blank">http://blog.superuser.com/2011/05/10/maximizing-the-lifetime-of-your-ssd/</a>
[42]<a href="http://serverfault.com/questions/356534/ssd-erase-block-size-lvm-pv-on-raw-device-alignment" target="_blank">http://serverfault.com/questions/356534/ssd-erase-block-size-lvm-pv-on-raw-device-alignment</a>
[43]<a href="http://rethinkdb.com/blog/page-alignment-on-ssds/" target="_blank">http://rethinkdb.com/blog/page-alignment-on-ssds/</a>
[44]<a href="http://rethinkdb.com/blog/more-on-alignment-ext2-and-partitioning-on-ssds/" target="_blank">http://rethinkdb.com/blog/more-on-alignment-ext2-and-partitioning-on-ssds/</a>
[45]<a href="http://rickardnobel.se/storage-performance-iops-latency-throughput/" target="_blank">http://rickardnobel.se/storage-performance-iops-latency-throughput/</a>
[46]<a href="http://www.brentozar.com/archive/2013/09/iops-are-a-scam/" target="_blank">http://www.brentozar.com/archive/2013/09/iops-are-a-scam/</a>
[47]<a href="http://www.acunu.com/2/post/2011/08/why-theory-fails-for-ssds.html" target="_blank">http://www.acunu.com/2/post/2011/08/why-theory-fails-for-ssds.html</a>
[48]<a href="http://security.stackexchange.com/questions/12503/can-wiped-ssd-data-be-recovered" target="_blank">http://security.stackexchange.com/questions/12503/can-wiped-ssd-data-be-recovered</a>
[49]<a href="http://security.stackexchange.com/questions/5662/is-it-enough-to-only-wipe-a-flash-drive-once" target="_blank">http://security.stackexchange.com/questions/5662/is-it-enough-to-only-wipe-a-flash-drive-once</a>
[50]<a href="http://searchsolidstatestorage.techtarget.com/feature/The-truth-about-SSD-performance-benchmarks" target="_blank">http://searchsolidstatestorage.techtarget.com/feature/The-truth-about-SSD-performance-benchmarks</a>
[51]<a href="http://www.theregister.co.uk/2012/12/03/macronix_thermal_annealing_extends_life_of_flash_memory/" target="_blank">http://www.theregister.co.uk/2012/12/03/macronix_thermal_annealing_extends_life_of_flash_memory/</a>
[52]<a href="http://www.eecs.berkeley.edu/~rcs/research/interactive_latency.html" target="_blank">http://www.eecs.berkeley.edu/~rcs/research/interactive_latency.html</a>
[53]<a href="http://blog.nuclex-games.com/2009/12/aligning-an-ssd-on-linux/" target="_blank">http://blog.nuclex-games.com/2009/12/aligning-an-ssd-on-linux/</a>
[54]<a href="http://www.linux-mag.com/id/8397/" target="_blank">http://www.linux-mag.com/id/8397/</a>
[55]<a href="http://tytso.livejournal.com/2009/02/20/" target="_blank">http://tytso.livejournal.com/2009/02/20/</a>
[56]<a href="https://wiki.debian.org/SSDOptimization" target="_blank">https://wiki.debian.org/SSDOptimization</a>
[57]<a href="http://wiki.gentoo.org/wiki/SSD" target="_blank">http://wiki.gentoo.org/wiki/SSD</a>
[58]<a href="https://wiki.archlinux.org/index.php/Solid_State_Drives" target="_blank">https://wiki.archlinux.org/index.php/Solid_State_Drives</a>
[59]<a href="https://www.kernel.org/doc/Documentation/block/cfq-iosched.txt" target="_blank">https://www.kernel.org/doc/Documentation/block/cfq-iosched.txt</a>
[60]<a href="http://www.danielscottlawrence.com/blog/should_i_change_my_disk_scheduler_to_use_NOOP.html" target="_blank">http://www.danielscottlawrence.com/blog/should_i_change_my_disk_scheduler_to_use_NOOP.html</a>
[61]<a href="http://www.phoronix.com/scan.php?page=article&amp;item=linux_iosched_2012" target="_blank">http://www.phoronix.com/scan.php?page=article&amp;item=linux_iosched_2012</a>
[62]<a href="http://www.velobit.com/storage-performance-blog/bid/126135/Effects-Of-Linux-IO-Scheduler-On-SSD-Performance" target="_blank">http://www.velobit.com/storage-performance-blog/bid/126135/Effects-Of-Linux-IO-Scheduler-On-SSD-Performance</a>
[63]<a href="http://www.axpad.com/blog/301" target="_blank">http://www.axpad.com/blog/301</a>
[64]<a href="http://en.wikipedia.org/wiki/List_of_solid-state_drive_manufacturers" target="_blank">http://en.wikipedia.org/wiki/List_of_solid-state_drive_manufacturers</a>
[65]<a href="http://en.wikipedia.org/wiki/List_of_flash_memory_controller_manufacturers" target="_blank">http://en.wikipedia.org/wiki/List_of_flash_memory_controller_manufacturers</a>
[66]<a href="http://blog.zorinaq.com/?e=29" target="_blank">http://blog.zorinaq.com/?e=29</a>
[67]<a href="http://www.gamersnexus.net/guides/956-how-ssds-are-made" target="_blank">http://www.gamersnexus.net/guides/956-how-ssds-are-made</a>
[68]<a href="http://www.gamersnexus.net/guides/1148-how-ram-and-ssds-are-made-smt-lines" target="_blank">http://www.gamersnexus.net/guides/1148-how-ram-and-ssds-are-made-smt-lines</a>
[69]<a href="http://www.tweaktown.com/articles/4655/kingston_factory_tour_making_of_an_ssd_from_start_to_finish/index.html" target="_blank">http://www.tweaktown.com/articles/4655/kingston_factory_tour_making_of_an_ssd_from_start_to_finish/index.html</a>
[70]<a href="http://www.youtube.com/watch?v=DvA9koAMXR8" target="_blank">http://www.youtube.com/watch?v=DvA9koAMXR8</a>
[71]<a href="http://www.youtube.com/watch?v=3s7KG6QwUeQ" target="_blank">http://www.youtube.com/watch?v=3s7KG6QwUeQ</a>
[72]<a href="https://www.usenix.org/conference/fast13/technical-sessions/presentation/zheng" target="_blank">Understanding the Robustness of SSDs under Power Fault, Zheng et al., 2013</a>—<a href="https://news.ycombinator.com/item?id=7047118" target="_blank">[discussion on HN]</a>
[73]<a href="http://lkcl.net/reports/ssd_analysis.html" target="_blank">http://lkcl.net/reports/ssd_analysis.html</a>—<a href="https://news.ycombinator.com/item?id=6973179" target="_blank">[discussion on HN]</a></span></p>



## 注 ##

本文由 `伯乐在线-熊铎` 翻译。已授权转载！

原文： http://codecapsule.com/2014/02/12/coding-for-ssds-part-2-architecture-of-an-ssd-and-benchmarking/

中文翻译： http://blog.jobbole.com/63881

下一部分：[第三部分](../../../../2016/02/25/coding-for-ssds-part-3-pages-blocks-and-the-flash-translation-layer/)在这。如果你比较忙，你可以直接去总结了所有其他部分内容的[第六部分](../../../../2016/02/25/coding-for-ssds-part-6-a-summary-what-every-programmer-should-know-about-solid-state-drives/)。
layout: post
title: disksim with ssdmodel 源码解析（七）：block的组织形式
date: 2016-3-18 21:23:22
tags: 
	- SSD
	- Disksim
comments: true
copyright: true
---


我们知道，在一个SSD中会有很多的 element (也就是chip)，每个element中会有很多个 plane，每个 plane 中会包含很多个 block。

在访问 SSD 时通常都会向设备发送需要访问的 block number ，因此了解 block 在 SSD 中的组织形式十分重要。在 disksim 的 ssdmodel 模块中一共定义了三种 block 在 SSD 中的组织形式。

在 `ssd.h` 文件中我们可以看到三条宏定义，分别是：

```C
#define PLANE_BLOCKS_CONCAT    1 
#define PLANE_BLOCKS_PAIRWISE_STRIPE 	2
#define PLANE_BLOCKS_FULL_STRIPE	3
```

<!--more-->

下面分别介绍这三种组织形式。

## **PLANE_BLOCKS_CONCAT** ##

这种组织形式每个 plane 上的 block 是顺序组织的，一个 plane 接着一个 plane ，整个是串在一起的，如下图所示：

![](/img/articles/disksim/blockrange1.jpg)


## **PLANE_BLOCKS_PAIRWISE_STRIPE** ##

这种组织形式比较奇特，是按对来组织的，每一对 plane 上的 block 是交错组织的，而对与对之间的组织是串联的，如下图所示：

![](/img/articles/disksim/blockrange2.jpg)


## **PLANE_BLOCKS_FULL_STRIPE** ##

这种方式下，每个 plane 之间的 block 是交错组织的，如下图所示：

![](/img/articles/disksim/blockrange3.jpg)


## **last but not the least** ##

关于不同的组织形式在垃圾回收的时候会用的到的，所以了解这个对理解后面垃圾回收部分的代码也是很有帮助的。

在 ssd 的元数据 `ssd_element_metadata` 中有一个  `char *free_block` 的属性。free_block 所代表的是一个连续的 bit 位，如果第 i 个 bit 为 1，那么表示该 element 的第 i 个 block 是 free 状态，否则就是处于正在使用状态。关于第 i 个 block 的 i 是按上面第一种方式来组织的，也就是每个 plane 的串联。所以对于给定的一个 block number 你只有知道 block 在 element 中的组织方式才能计算出他是第几个 block。

在 `ssd_clean.c` 中提供了 `ssd_block_to_bitpos()` 这个函数，你传入 ssd 和 blkno 它根据当前 ssd 中 block 的组织形式返回出 blkno 在当前 element 中对应的编号是多少。然后调用 `ssd_bit_on()` 可以查看当前 block 处于 free 还是 in use 状态。
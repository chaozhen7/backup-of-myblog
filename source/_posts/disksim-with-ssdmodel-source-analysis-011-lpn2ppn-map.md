layout: post
title: disksim with ssdmodel 源码解析（十一）：地址映射
date: 2016-3-23 10:23:22
tags: 
	- SSD
	- Disksim
comments: true  
---

 SSD 中的逻辑地址到物理地址的映射方式有很多中，这里主要是介绍 ssdmodel 中的映射方式。

由于 SSD 与主机的接口使用的还是以前的 HDD 的接口，所以操作系统给 SSD 发出请求时仍然是以扇区(sector，512B)为单位。例如一条trace数据为 (0.0,0,512,8,0)，表示的是要对第512个扇区进行操作。由于 SSD 读写操作的基本单位时页(page, 4KB)，所以操作系统给 SSD 发请求时的地址通常都是 8 的整数倍。

SSD 地址映射最简单的方式就是以页为单位一对一映射(类似cache的全相联映射)，当然这是很费内存的。在 ssdmodel 中为了减少内存开销，通常是以一个 element 为单位在其内部进行以 page 为单位的一一映射。 在 element(chip) 的元数据里面有一个 `lba_table[]` 的数组，表示的逻辑页到物理页的映射。比如 `lab_table[lpn] = ppn `，代表的是该 element 上的逻辑页号 lpn 对应着该 element 上的物理页号 ppn。

<!--more-->

## **sector 号到逻辑地址的转换** ##

对于操作系统发过的以 sector 为单位的地址是如何转化得到其在 SSD 逻辑地址的，是使用下面的公式来计算的：


$ element\\_ no =  \frac{blkno}{(element\\_stride\\_pages*page\\_size)}\\%nelement $
 
$  apn = \frac{blkno}{page\\_size} $

$  lpn = \frac {apn - (apn \\% (element\\_stride\\_pages * nelements))}{nelements} + (apn \\% element\\_stride\\_pages) $


上面的 `element_no` 代表着这个请求将会被映射到 SSD 的哪个 element 上，`lpn`(logical page number) 代表这个请求地址对应的 element 上的逻辑页号。上面的公式分别在 `ssd_timing.c` 的 `ssd_choose_element()` 函数和 `ssd.c` 的 `ssd_logical_pageno()`函数中。


### **符号的说明：** ###


`element_stride_pages`： 通常为1或2

`page_size`: 一个页中包含 sector 的数量，通常为 8

`nelement`: ssd 中 element 的总数量

注：本文中所有的除运算都表示整除。

### **简化** ###

如果 `element_stride_pages` 为1，那么上面的公式会得到一个简化，如下：

$ element\\_ no =  \frac{blkno}{page\\_size}\\%nelement $
 

$  lpn = \frac {blkno}{nelements * page\\_size}$


## **逻辑页到物理页的映射** ##

在每个 element 的元数据中都有一个 `lba_table[]` 表来维持该 element 中的逻辑地址到物理地址的映射关系，下标表示逻辑地址，值表示物理地址。

在每个 block 的元数据中还有一个 `page[]` 表，表的大小等于 block 中 page 的数量，page[] 的值表示物理页所对应的逻辑页是多少。下标表示该 page 在 block 中的序号，值表示该 page 的逻辑页号。

当一个写请求到达时，如果它的逻辑页地址为 lpn，写入的物理页为 active page，那么会令 `lba_table[lpn] = active page`。

当下一个写请求到达时，如果还是和上一个写请求的逻辑页号一样，首先会把 lpn 对应的物理块的 page[] 值置为 -1，再更新 lba_table[lpn] 到当前的 active page。



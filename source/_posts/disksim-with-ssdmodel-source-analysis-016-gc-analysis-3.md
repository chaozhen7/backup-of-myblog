layout: post
title: disksim with ssdmodel 源码解析（十六）：GC 分析(三)
date: 2016-3-29 14:23:22
tags: 
	- SSD
	- Disksim
comments: true
copyright: true
---


今天这篇文章主要介绍什么时候会触发 GC。

判断是否需要进行 GC 操作是分为两种不同的情况来判断的：有 copyback 和没有 copyback 。


在 `ssd_clean.c` 中的 `ssd_clean_element()` 函数中会根据是否有 copyback 来调用不同的函数区执行 GC 操作。

下面分别介绍在这两种情况下，达到什么条件时会触发 GC 操作。

<!--more-->

## 无 copyback ##

这种情况是最简单的，直接判断该 element 上的空闲块是否达到了预先所设定的阈值。

在  `ssd_clean_element_no_copyback()` 中会调用 `ssd_start_cleaning(int plane_num, int elem_num, ssd_t *s)` 函数。`ssd_start_cleaning()` 函数的内容如下：

```C
// invoke cleaning when the number of free blocks drop below a
// certain threshold (in a plane or an element).
int ssd_start_cleaning(int plane_num, int elem_num, ssd_t *s)
{
    if (plane_num == -1) {
        unsigned int low = (unsigned int)LOW_WATERMARK_PER_ELEMENT(s);
        return (s->elements[elem_num].metadata.tot_free_blocks <= low);
    } else {
        int low = (int)LOW_WATERMARK_PER_PLANE(s);
        return (s->elements[elem_num].metadata.plane_meta[plane_num].free_blocks <= low);
    }
}
```

因为在 `ssd_clean_element_no_copyback()` 中调用 `ssd_start_cleaning()` 时传入的第一个参数为 -1 所以这个时候在 `ssd_start_cleaning()` 中实际上只执行了两行代码：

```c
unsigned int low = (unsigned int)LOW_WATERMARK_PER_ELEMENT(s);
return (s->elements[elem_num].metadata.tot_free_blocks <= low);
```

low 在这里表示的就是每个 element 上的 blocks 的数量乘以最低空闲块的百分比。

所以这种方式下，只要某个 element 上的空闲块低于阈值，就会判定为需要垃圾回收。


## 有 copyback ##

这种判定方式稍微复杂些。是在 `ssd_clean_element_copyback(int elem_num, ssd_t *s)` 函数中判定的。

当我们需要判断某个 element 上是否需要进行垃圾回收时，首先会依次遍历这个 element 上的每一个并行单元（由几个plane组成），对于每个并行单元会依次遍历这个并行单元里的每个 plane 并调用 `ssd_start_cleaning()` 这个函数(这个时候需要传入 plane_num，跟上面的不同)，如果这个 plane 上的空闲 block 低于预先设定的阈值，那么就标记这个 plane 表示需要进行垃圾回收。

这种情况下，一个 element 上的每个并行单元里的每一个 plane 都要检查判断，但是一个并行单元里只要检查出一个 plane 需要进行垃圾回收其他的就不用检查了。只要有一个并行单元里检查除了需要垃圾回收的 plane，那么这个 element 就需要进行进行垃圾回收。

因为每个并行单元之间的写是并行的，所以在这种情况下做垃圾回收时，需要对每个并行单元都检查，从每个并行单元中都要选出一个 plane 进行垃圾回收（如果这个并行单元不需要垃圾回收就可以不用选），以保证下次的并行写能够同时对这几个并行单元进行写操作。

流程图如下：

![有copyback时判定是否触发GC的流程图](/img/articles/disksim/gc3.jpg)
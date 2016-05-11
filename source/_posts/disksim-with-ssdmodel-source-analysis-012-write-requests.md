layout: post
title: disksim with ssdmodel 源码解析（十二）：写操作分析
date: 2016-3-23 12:23:22
tags: 
	- SSD
	- Disksim
comments: true
copyright: true
---

这里主要介绍的如何写入具体的一个页，而不是介绍写请求的整个流程(这个后面会介绍)。

回顾前面的知识，在 ssdmodel 在每个 element 的元数据中都有一个 `lba_table[]` 表来维持该 element 中的逻辑地址到物理地址的映射关系，下标表示逻辑地址，值表示物理地址。

在每个 block 的元数据中还有一个 `page[]` 表，表的大小等于 block 中 page 的数量，page[] 的值表示物理页所对应的逻辑页是多少。下标表示该 page 在 block 中的序号，值表示该 page 的逻辑页号。

当一个写请求到达时，如果它的逻辑页地址为 lpn，写入的物理页为 active page，那么会令 `lba_table[lpn] = active page`。

当下一个写请求到达时，如果还是和上一个写请求的逻辑页号一样，首先会把 lpn 对应的物理块的 page[] 值置为 -1，再更新 lba_table[lpn] 到当前的 active page。

介绍具体的写操作之前先介绍下相关的 element 和 block 中的元数据的属性(**仅列举本文中提到的，其他的忽略**)。

<!--more-->

```C
typedef struct _ssd_element_metadata {
    int *lba_table;       // a table mapping the lba to the physical pages on the chip.

    unsigned int active_page;       // this points to the next page to write inside an active block.

    plane_metadata plane_meta[SSD_MAX_PLANES_PER_ELEM];

    block_metadata *block_usage;    // contains the number of valid pages in each block.
                                    // we also store the valid page numbers here. this is useful
                                    // during cleaning.

    unsigned int bsn;               // block sequence number for this ssd element

    int plane_to_write;             // which plane to write next?

} ssd_element_metadata;

```

```C
typedef struct _plane_metadata {

    int valid_pages;                // num of valid pages (note that a block might not
                                    // be free but not all its pages should be valid)

    unsigned int active_page;       // this points to the next page to write inside an
                                    // active block.

    int block_alloc_pos;            // block allocation position in a plane

} plane_metadata;
```
```C
typedef struct _block_metadata {
    int         block_num;              //
    int         plane_num;              // the plane to which this block belongs to

    int         *page;                  // size of the array = pages_per_block
                                        // holds all the valid logical page numbers present
                                        // in a block

    int         num_valid;              // represents the no of valid pages in the block (that is,
                                        // the no of non '-1' entries in the above page array)
                                        // don't use it as an index to add pages in the above array.

    int         state;                  // state of the block
                                        // a block could be in one of the following states:
                                        // CLEAN - when the block is erased and clean to take new writes
                                        // INUSE - when its pages are being filled
                                        // SEALED - when all the free pages are written and the block
                                        // is sealed for future writes

    unsigned int    bsn;                // block sequence number (version number for blocks)
} block_metadata;
```

### **写操作的步骤** ###

写操作调用的是 `ssd_timing.c` 中的 `_ssd_write_page_osr(ssd_t *s, ssd_element_metadata *metadata, int lpn)` 函数，传入的参数分别是：写入的 ssd 的指针 `s`，写入的 element 的元数据指针 `metadata`，写入的逻辑页号 `lpn`。 返回的是该写操作所耗的时间。

**step1: 获取当前 active page 的信息**
```C
unsigned int active_page = metadata->active_page;   //下一个将要写入的页
unsigned int active_block = SSD_PAGE_TO_BLOCK(active_page, s);  // active page 所在的block
unsigned int pagepos_in_block = active_page % s->params.pages_per_block;   // active page 在某个block中的序号
unsigned int active_plane = metadata->block_usage[active_block].plane_num;  // active page 所在plane
```

**step2: 获取 lpn 所对应的物理页的信息**
```C
unsigned int prev_page = metadata->lba_table[lpn];    // 该逻辑页之前所对应的物理页
unsigned int prev_block = SSD_PAGE_TO_BLOCK(prev_page, s);  // 逻辑页所在的物理块
unsigned int pagepos_in_prev_block = prev_page % s->params.pages_per_block;  //逻辑页在物理块上的序号
unsigned int prev_plane = metadata->block_usage[prev_block].plane_num;  // 逻辑页所在的plane
```

**step3: 更新旧的物理页的状态**
```C
metadata->block_usage[prev_block].page[pagepos_in_prev_block] = -1;  //将之前的页标记为无效
metadata->block_usage[prev_block].num_valid --;   // 将之前块的有效页数减1
metadata->plane_meta[prev_plane].valid_pages --;  // 将之前plane的有效页数减1
```

**step4: 更新映射表**
```C
 metadata->lba_table[lpn] = active_page;
```

**step5: 更新的物理页信息**
```C
metadata->block_usage[active_block].page[pagepos_in_block] = lpn;  // 标记block中第对应的页的物理号位lpn
metadata->block_usage[active_block].num_valid ++;    //对应块的有效页数加1
metadata->plane_meta[active_plane].valid_pages ++;  //对应plane中的有效页数加1
```

**step7： 更新 active page**
```C
metadata->active_page = active_page + 1;
metadata->plane_meta[active_plane].active_page = metadata->active_page;
```

**step6: 判断是否已经写入到最后一个页**
```C
	// if this is the last data page on the block, let us write the summary page also
	if (ssd_last_page_in_block(metadata->active_page, s)) {
		cost += ssd_data_transfer_cost(s, SSD_SECTORS_PER_SUMMARY_PAGE);  // 传输一个 summary page data 的时间开销
		cost += s->params.page_write_latency;  //写入一个 summary page data 的时间开销

        // seal the last summary page. since we use the summary page
        // as a metadata, we don't count it as a valid data page.
        metadata->block_usage[active_block].page[s->params.pages_per_block - 1] = -1;   // 标记当前块的最后一个页无效
        metadata->block_usage[active_block].state = SSD_BLOCK_SEALED;   //标记当前块已经全部写完
    }
```



## 写在后面 ##

ssdmodel 在初始化 ssd 的时候会将每个 block 的 lba_table[] 表都初始化，逻辑地址与物理地址的对应都是依次递增。

所以在对某个 page 进行第一次写时，程序会认为该块已经被写入了，会在 block 的元数据中标记 page[] 所对应的值为 -1 ，然后再更新 element 的 lba_table[] 表，其实这个时候这个 page 明明是新的，却被误以为是无用的。不知道是我理解错了，还是作者认为做出这种假设是合理的。


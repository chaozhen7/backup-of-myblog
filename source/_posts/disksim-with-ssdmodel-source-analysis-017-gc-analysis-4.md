layout: post
title: disksim with ssdmodel 源码解析（十七）：GC 分析(四)
date: 2016-3-30 19:23:22
tags: 
	- SSD
	- Disksim
comments: true
copyright: true
---

前面介绍的大多都是 GC 操作的判断逻辑之类的，今天主要介绍在 ssdmodel 中是如何具体实现 GC 操作的。

在 ssdmodel 中实现 GC 的算法大概分为两种（准确的说应该是三种吧）。
- **随机算法**：在回收某个 element(或plane)的时候随机在其中选择一个符合要求的 block 进行回收。
- **贪心算法**：在回收某个 element(或plane)的时候从其中选择有效页最少的 block 进行回收。（这种算法又分为考虑 wear-aware 和不考虑 wear-aware 的情况）

除了这两种(或者三种吧)算法外，在 ssdmodel 实现中有 copyback 和没 copyback 的回收也是不一样的。

<!--more-->

下面主要介绍在没有 copyback 情况下利用贪心算法如何来实现 GC。


结合前面的理一理垃圾回收的操作的整个流程吧。

`ssd_activate_elem()` ---> `ssd_invoke_element_cleaning()` ---> `_ssd_invoke_element_cleaning()` ---> `ssd_clean_element()` ---> `(if no copyback)` ---> `ssd_clean_element_no_copyback()` ---> `(if达到gc条件)` ---> `(if贪心算法)` ---> `ssd_clean_blocks_greedy()`


好了，今天就主要介绍 `ssd_clean_blocks_greedy(int plane_num, int elem_num, ssd_t *s)` 中是如何利用贪心算法来进行垃圾回收的。如果这里的 plane_num=-1 则表示把该 element 当成一个整体来进行垃圾回收，否则的话就是把这个 element 上编号为 plane_num 的 plane 当成一个整体进行回收。


### **step1. 建立 table** ###

首先会调用 `ssd_build_usage_table(elem_num, s)` 这个函数来建立一个 `table`，tabel 可以看成是一个数组吧，数组大小等于一个 block 中 page 的数量，数组的成员为 `usage_table` 结构体。程序中对 usage_table 的定义如下：

```C
typedef struct _usage_table {
    int len;   // 有效页为某个值的 block 的数量
    int temp;
    int *block; // 可以把这个看成是一个数组block[],数组的大小是动态的，等于len值。其中保存的是特定的blkno
} usage_table;
```

如何来构建这个 table[] 可以看看 `ssd_build_usage_table()` 这个函数的源码。这里只介绍下 table[] 中保存的信息，其实也就是 `ssd_build_usage_table()` 这个函数所实现的功能。

这里的这个 table 其实也可以理解成是一个 hash 表吧，键值就是这个有效页的数量。比如在该 element 中有效页数量为5的 block 有10个，那么 table[5].len=10，table[5] 的 block[] 里面保存的就是这10个 block 的 blkno。

好像上面的还是有点绕，那么再举个例子，假设当前 element 中一共有 8 个block 对应的 blkno 分别为 0-7，拥有的有效页分数分别为:10, 20, 10, 30, 45, 50, 10, 20。那么 table 的值应该就是如下面这样：

table[10].len = 3;	table[10].block={0,2,6}
table[20].len = 2;	table[20].block={1,7}
table[30].len = 1;	table[30].block={3}
table[45].len = 1;	table[45].block={4}
table[50].len = 1;	table[50].block={5}
table[其他].lent = 0;	table[其他].block=NULL


### **step2. 计算平均剩余生命值 avg_lifetime** ###

这里会调用 `ssd_compute_avg_lifetime(plane_num, elem_num, s)` 这个函数来计算当前 element(或plane) 中 block 的平均剩余生命值(剩余的可擦除次数)，如果这里 plane_num=-1 那么就是计算整个 element 中 block 的平均剩余生命值，否则就是计算这个 element 上指定的 plane 中 block 的平均剩余生命值。


### **step3. 执行垃圾回收操作** ###

这一步主要是依次遍历 table 数组，从有效页数量最少的开始（也就是从 table[0] 开始），每次遍历一个 table 元素的时候都会依次遍历它的 block[] 属性中的每一个 block，对它进行回收。每完成对其中一个 block 的回收后都要调用 `ssd_stop_cleaning()` 判断当前 element(或plane) 中的空闲块是否达到预先设定的值，如果达到了就不再行垃圾回收了。

遍历 table 用的是一个 for 循环，i从0一直到最大值(一个 block 中 page 的数量)，表示优先回收有效页少的 block。对于 table[i]，也是用一个for循环依次遍历一个 usage_table 元素中的 block[] 属性,也就是遍历 table[i] 中的 block[]，j从0到 table[i].len。

对于最内层的循环，每次其实是选取出来了一个 block 进行回收，这个回收过程大致分为如下几个步骤：

(1) 对于 plane_num!=-1 的情况，每一个将要进行回收的 block 都要判断该 block 是否在该 plane 上，如果不在的话就重新选择下一个block，如果 plane_num=-1，则不需要此步骤的判断。

(2) 判断该 block 是否已经不可用了，也就是判断该 block 的剩余生命值block_life (剩余可擦出次数）是否为0。如果为0的话就重新选择下一个 block，否则继续下一步。

(3) 调用 `ssd_can_clean_block()` 判断该 block 是否处于可回收状态，如果不是则重新选择下一个block，如果是继续下一步。

(4) 这一步主要是针对需要考虑 wear-aware 情况的，如果不需要考虑 wear-aware ，可以直接跳到下一步。这一步主要判断当前 block 的剩余生命值是否不低于平均生命值avg_lifetime的某个百分比SSD_LIFETIME_THRESHOLD_X(程序约定好的)。如果不低于，直接进入到下一步。如果高于的话就以一定的概率放弃回收这个块，继续选择下一个待回收的block。

这个一定的概率是多大呢，block_life/avg_lifetime/SSD_LIFETIME_THRESHOLD_X，这么大(/ 表示除以，不是整除，带小数的那种除)。以一定的概率放弃回收这个块是怎么实现的呢，抛出一个随机数如果大于这个概率值就放弃回收这个块，继续选择下一个block，否则进入到下一步回收这个block。

(5) 经过层层筛选，如果这个 block 还能保留到现在，那就回收它吧，就是它了。调用 `_ssd_clean_block_fully()` 回收这个 block。

(6) 调用 `ssd_stop_cleaning()` 判断当前 element(或plane)中的有效块是否达到了程序预先设定的值，如果是的话跳出循环，停止垃圾回收，否则继续循环。因为这里有两层循环，所以 `ssd_stop_cleaning()` 这个函数在两个循环的末尾都要被调用，从而跳出两层循环。










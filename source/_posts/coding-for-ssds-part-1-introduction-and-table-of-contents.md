layout: post
title: 为 SSD 编程（1）：简介和目录
date: 2016-2-25 14:20:22
tags: 
	- SSD
comments: true
---

**简介**

我想为我的[键值对存储项目](http://codecapsule.com/2012/11/07/ikvs-implementing-a-key-value-store-table-of-contents/)弄一个固态硬盘（SSD）最佳存储解决方案。为此，我必须确保我完全了解SSD是如何工作的，这样就可以优化我的hash表实例来适合SSD的内部特征。网上有很多不完全和相悖的的信息，找到关于SSD的可靠信息并不简单。为了找到适当的文献和基准以说服自己，我必须要进行大量的阅读。如果我要为SSD编程，我需要知道我在做什么。

<!--more-->

研究完之后我搞明白了，相信将我所得到的结论向大家分享会有用的。我的意图是将所有的可用信息转化为实用知识。最后我写了一篇30页的文章，但它不太适合发布在博客上。因此我决定将这篇文章分为数个可被独立消化的数个逻辑部分。而目录在本文的下方。

[第六部分](../../../../2016/02/25/coding-for-ssds-part-6-a-summary-what-every-programmer-should-know-about-solid-state-drives/)有最为显著的贡献，这一部分是整个“为SSD编程”文章系列的总结，我相信那些心急的程序员最为喜欢。这个总结涵盖了SSD的基本知识以及所有推荐的实现固态硬盘的最佳读写性能的访问模式。

另外一个重要的细节是，“为SSD编程”与我的键值对存储项目（[IKVS series](http://codecapsule.com/2012/11/07/ikvs-implementing-a-key-value-store-table-of-contents/)）之间是相互独立的，因此并不需要IKVS文章中的知识。我正打算写一篇关于IKVS series的文章，内容是关于 如何用hash表来实现利用SSD的内部特征，不过我还没有具体的发布时间。

我唯一的遗憾就是，我还没有写任何代码，以证明我建议的访问模式确实是最好的。但是即使有这些代码，我也需要在大量不同的固态硬盘上测试性能基准，这将消耗我所不能承受的大量的时间和金钱。我仔细列举了我引用的文章，如果你认为我的建议不对，请留下评论指明。当然，如果你有什么疑问或想在任何方面做出贡献，请随意留下评论。

最后，不要忘了订阅电子报，以便在每当有新文章发表在Code Capsule上时能够收到通知。订阅面板在博客的右上角。

目录

第一部分：简介和目录

[第二部分：SSD的架构和基准](../../../../2016/02/25/coding-for-ssds-part-2-architecture-of-an-ssd-and-benchmarking/)

1. SSD的结构
1.1 NAND闪存单元
1.2 SSD的组织
1.3 制造过程

2. 基准和性能指标
2.1 基本基准
2.2 预处理
2.3 任务负载和指标

[第三部分：页、块、和闪存转换层](../../../../2016/02/25/coding-for-ssds-part-3-pages-blocks-and-the-flash-translation-layer/)

3. 基本操作
3.1 读、写、擦除
3.2 写入的例子
3.3 写入放大
3.4 耗损均衡

4. 闪存转换层（FTL）
4.1 FTL的必要性
4.2 逻辑块映射
4.3 产业状态的备注
4.4 垃圾回收

[第四部分：高级功能和内部并行](../../../../2016/02/25/coding-for-ssds-part-4-advanced-functionalities-and-internal-parallelism/)

5. 高级功能
5.1 TRIM
5.2 预留空间
5.3 完全抹除
5.4 本地命令队列(NCQ)
5.5 掉电保护

6. SSD的内部并行
6.1 被限制的I/O总线带宽
6.2 多层并行
6.3 块的集群

[第五部分：访问模式和系统优化](../../../../2016/02/25/coding-for-ssds-part-5-access-patterns-and-system-optimizations/)

7. 访问模式
7.1 定义顺序和随机I/O操作
7.2 写入
7.3 读出
7.4 合并读写

8. 系统操作
8.1 分区对齐
8.2 文件系统参数
8.3 操作系统I/O调度
8.4 交换空间
8.5 临时文件

[第六部分：总结——每个程序员都应该了解的固态硬盘知识](../../../../2016/02/25/coding-for-ssds-part-6-a-summary-what-every-programmer-should-know-about-solid-state-drives/)



</br>
## 注 ##

本文由 `伯乐在线-熊铎` 翻译。已授权转载！

原文：http://codecapsule.com/2014/02/12/coding-for-ssds-part-1-introduction-and-table-of-contents/

中文翻译：http://blog.jobbole.com/63520/

下一部分：[第二部分](../../../../2016/02/25/coding-for-ssds-part-2-architecture-of-an-ssd-and-benchmarking/)在这。如果你比较忙，你可以直接去总结了所有其他部分内容的[第六部分](../../../../2016/02/25/coding-for-ssds-part-6-a-summary-what-every-programmer-should-know-about-solid-state-drives/)。
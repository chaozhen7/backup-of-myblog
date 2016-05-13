layout: post
title: Memcached 学习（四）：统计命令介绍
date: 2016-5-11 16:55:22
tags:
	- Memcached
	- 数据库
comments: true
toc: true
---

## 1. stats 命令

Memcached stats 命令用于返回统计信息例如 PID(进程号)、版本号、连接数等。

当我们输入 `stats` 命令后。可以看到如下信息（仅列出部分）：

<!--more-->
> STAT pid 1162
STAT uptime 5022
STAT time 1415208270
STAT version 1.4.14
...
内容太多，省略大部分
...
STAT bytes 142
STAT curr_items 2
STAT total_items 6
STAT evictions 0
STAT reclaimed 1
END

这里显示了很多状态信息，下边选取部分状态项解释：
- pid： memcache服务器进程ID
- uptime：服务器已运行秒数
- time：服务器当前Unix时间戳
- version：memcache版本
- pointer_size：操作系统指针大小
- rusage_user：进程累计用户时间
- rusage_system：进程累计系统时间
- curr_connections：当前连接数量
- total_connections：Memcached运行以来连接总数

## 2. stats items 命令

Memcached stats items 命令用于显示各个 slab 中 item 的数目和存储时长(最后一次访问距离现在的秒数)。

当输入 `stats items` 后可以看到：
> stats items
STAT items:1:number 1
STAT items:1:age 7
STAT items:1:evicted 0
STAT items:1:evicted_nonzero 0
STAT items:1:evicted_time 0
STAT items:1:outofmemory 0
STAT items:1:tailrepairs 0
STAT items:1:reclaimed 0
STAT items:1:expired_unfetched 0
STAT items:1:evicted_unfetched 0
END

## 3. stats slabs 命令

Memcached stats slabs 命令用于显示各个slab的信息，包括chunk的大小、数目、使用情况等。

## 4. stats sizes 命令

Memcached stats sizes 命令用于显示所有item的大小和个数。

该信息返回两列，第一列是 item 的大小，第二列是 item 的个数。

比如输入 `stats sizes` 会输出：
> STAT 64 4
END

## 5. flush_all 命令

Memcached flush_all 命令用于清理缓存中的所有 key=>value(键=>值) 对。

该命令提供了一个可选参数 time，用于在制定的时间后执行清理缓存操作。

语法：
```
flush_all [time] [noreply]
```





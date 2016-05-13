layout: post
title: redis 学习（三）：redis 数据类型
date: 2016-5-12 11:20:22
tags: 
	- Redis
	- 数据库
comments: true
toc: true
---

Redis支持5种数据类型: string（字符串），hash（哈希），list（列表），set（集合）及zset(sorted set：有序集合)。

<!--more-->

## 1. String（字符串）

string 是redis最基本的类型，你可以理解成与Memcached一模一样的类型，一个key对应一个value。

Redis的字符串是字节序列。在Redis中字符串是二进制安全的，这意味着他们有一个已知的长度，是没有任何特殊字符终止决定的，所以可以存储任何东西，比如图片或者序列化的对象。

string 最大长度可达 512MB，也就是一个键最大能存储512MB。

## 2. Hash（哈希）

Redis hash 是一个键值对集合。

Redis hash 是一个 string 类型的 field 和 value 的映射表，hash 特别适合用于存储对象。

比如：
```
hmset user:1 username chen password 123456 points 20
```
如果接着输入 `hgetall user:1` 可以得到如下内容
> 1) "username"
2) "chen"
3) "password"
4) "123456"
5) "points"
6) "20"

以上实例中 hash 数据类型存储了包含用户脚本信息的用户对象。 实例中我们使用了 Redis `hmset`, `hgetall` 命令，user:1 为键值。

每个 hash 可以存储 $2^{32}-1$ 个键值对（40多亿）。

## 3. List（列表）

Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素导列表的头部（左边）或者尾部（右边）。

如果连续输入下面的三条命令：
```
lpush ch chen1
lpush ch chen2
lpush ch chen3
```
再执行命令 `lrange ch 0 10` 我们可以得到：
> 1) "chen3"
2) "chen2"
3) "chen1"

列表最多可存储 $2^{32}-1$ 个元素 (4294967295, 每个列表可存储40多亿)。

## 4. Set（集合）

Redis的Set是string类型的无序集合。

集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)。

添加一个string元素到,key对应的set集合中，成功返回1，如果元素已经在集合中返回0，key对应的set不存在返回错误。

如果连续输入下面的命令：
```
sadd cc chen1
sadd cc chen2
sadd cc chen2
```
再输入命令 `smembers cc` 可得到如下内容：
> 1) "chen1"
> 2) "chen2"

以上实例中 chen2 添加了两次，但根据集合内元素的唯一性，第二次插入的元素将被忽略。

集合中最大的成员数为 $2^{32}-1$ (4294967295, 每个集合可存储40多亿个成员)。

## 5. Zset(有序集合)

Redis zset 和 set 一样也是string类型元素的集合,且不允许重复的成员。

不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。

zset的成员是唯一的,但分数(score)却可以重复。

如果连续输入下面的命令：
```
zadd hh 0 chen1
zadd hh 3 chen2
zadd hh 2 chen3
zadd hh 1 chen3
```
再输入命令 `zrangebyscore hh 0 10` 可得到如下内容：
> 1) "chen1"
2) "chen3"
3) "chen2"





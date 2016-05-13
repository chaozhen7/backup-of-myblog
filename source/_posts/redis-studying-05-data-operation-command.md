layout: post
title: redis 学习（五）：数据操作相关命令
date: 2016-5-12 13:20:22
tags: 
	- Redis
	- 数据库
comments: true
toc: true
---

前面介绍了 redis 中的基本数据类型有五种: string（字符串），hash（哈希），list（列表），set（集合）及zset(sorted set：有序集合)。

下面将分别介绍操作这五中基本数据类型时所涉及到的命令。

<!--more-->


## 1. String(字符串)

|命令| 描述|
|---|---|
|   **set** key value  |  设置指定 key 的值  |
|  	**get** key  |  获取指定 key 的值。  |
|  	**getrange** key start end  | 返回 key 中字符串值的子字符  |
|  	**getset** key value  |将给定 key 的值设为 value ，并返回 key 的旧值(old value)。  |
|  	**getbit** key offset  |对 key 所储存的字符串值，获取指定偏移量上的位(bit)。  |
|	**mget** key1 [key2..]  |获取所有(一个或多个)给定 key 的值。  |
|	**setbit** key offset value  |对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit)。  |
|	**setex** key seconds value  |将值 value 关联到 key ，并将 key 的过期时间设为 seconds (以秒为单位)。  |
|	**setnx** key value  |只有在 key 不存在时设置 key 的值。  |
|	**setrange** key offset value  |用 value 参数覆写给定 key 所储存的字符串值，从偏移量 offset 开始。  |
|	**strlen** key  |返回 key 所储存的字符串值的长度。  |
|	**mset** key value [key value ...]  |同时设置一个或多个 key-value 对。  |
|	**msetnx** key value [key value ...]  |同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在。  |
|	**psetex** key milliseconds value  |这个命令和 SETEX 命令相似，但它以毫秒为单位设置 key 的生存时间，而不是像 SETEX 命令那样，以秒为单位。  |
|	**incr** key  |将 key 中储存的数字值增一。  |
|	**incrby** key increment  |将 key 所储存的值加上给定的增量值（increment） 。  |
|	**incrbyfloat** key increment  |将 key 所储存的值加上给定的浮点增量值（increment） 。  |
|	**decr** key  |将 key 中储存的数字值减一。  |
|	**decrby** key decrement  |key 所储存的值减去给定的减量值（decrement） 。  |
|	**append** key value  |  如果 key 已经存在并且是一个字符串， APPEND 命令将 value 追加到 key 原来的值的末尾。

## 2. Hash(哈希)

|命令| 描述|
|---|---|
|   **hdel** key field2 [field2]    |  删除一个或多个哈希表字段    |
|	**hexists** key field    | 查看哈希表 key 中，指定的字段是否存在。    |
|	**hget** key field    | 获取存储在哈希表中指定字段的值/td>    |
|	**hgetall** key    | 获取在哈希表中指定 key 的所有字段和值    |
|	**hincrby** key field increment    | 为哈希表 key 中的指定字段的整数值加上增量 increment 。    |
|	**hincrbyfloat** key field increment    | 为哈希表 key 中的指定字段的浮点数值加上增量 increment 。    |
|	**hkeys** key    | 获取所有哈希表中的字段    |
|	**hlen** key    | 获取哈希表中字段的数量    |
|	**hmget** key field1 [field2]    | 获取所有给定字段的值    |
|	**hmset** key field1 value1 [field2 value2 ]    | 同时将多个 field-value (域-值)对设置到哈希表 key 中。    |
|	**hset** key field value    | 将哈希表 key 中的字段 field 的值设为 value 。    |
|  **hsetnx** key field value    | 只有在字段 field 不存在时，设置哈希表字段的值。    |
|	**hvals** key    | 获取哈希表中所有值    |
|	**hscan** key cursor \[MATCH pattern\] \[COUNT count\]    | 迭代哈希表中的键值对。 |

## 3. List(列表)

|命令| 描述|
|---|---|
|   **blpop** key1 [key2 ] timeout   |  移出并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。   |
|	**brpop** key1 [key2 ] timeout   |  移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。   |
|	**brpoplpush** source destination timeout   |  从列表中弹出一个值，将弹出的元素插入到另外一个列表中并返回它； 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。   |
|	**lindex** key index   |  通过索引获取列表中的元素   |
|	**linsert** key BEFORE 或 AFTER pivot value   |  在列表的元素前或者后插入元素   |
|	**llen** key   |  获取列表长度   |
|	**lpop** key   |  移出并获取列表的第一个元素   |
|	**lpush** key value1 [value2]   |  将一个或多个值插入到列表头部   |
|	**lpushx** key value   |  将一个或多个值插入到已存在的列表头部   |
|	**lrange** key start stop   |  获取列表指定范围内的元素   |
|	**lrem** key count value   |  移除列表元素   |
|	**lset** key index value   |  通过索引设置列表元素的值   |
|	**ltrim** key start stop   |  对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。   |
|	**rpop** key   |  移除并获取列表最后一个元素   |
|	**rpoplpush** source destination   |  移除列表的最后一个元素，并将该元素添加到另一个列表并返回   |
|	**rpush** key value1 [value2]   |  在列表中添加一个或多个值   |
|	**rpushx** key value   |  为已存在的列表添加值   |

## 4. Set(集合)

|命令| 描述|
|---|---|
|  **sadd** key member1 [member2]   |向集合添加一个或多个成员   |
|	**scard** key   |获取集合的成员数   |
|	**sdiff** key1 [key2]   |返回给定所有集合的差集   |
|	**sdiffstore** destination key1 [key2]   | 返回给定所有集合的差集并存储在 destination 中   |
|	**sinter** key1 [key2]   | 返回给定所有集合的交集   |
|	**sinterstore** destination key1 [key2]   | 返回给定所有集合的交集并存储在 destination 中   |
|	**sismember** key member   | 判断 member 元素是否是集合 key 的成员   |
|	**smembers** key   | 返回集合中的所有成员   |
|	**smove** source destination member   | 将 member 元素从 source 集合移动到 destination 集合   |
|	**spop** key   | 移除并返回集合中的一个随机元素   |
|	**srandmember** key [count]   | 返回集合中一个或多个随机数   |
|	**srem** key member1 [member2]   | 移除集合中一个或多个成员   |
|	**sunion** key1 [key2]   | 返回所有给定集合的并集   |
|	**sunionstore** destination key1 [key2]   | 所有给定集合的并集存储在 destination 集合中   |
|	**sscan** key cursor \[MATCH pattern\] \[COUNT count\]   | 迭代集合中的元素   |

## 5. sorted set(有序集合)

|命令| 描述|
|---|---|
|	**zadd** key score1 member1 [score2 member2]   | 向有序集合添加一个或多个成员，或者更新已存在成员的分数   |
|	**zcard** key   | 获取有序集合的成员数   |
|	**zcount** key min max   | 计算在有序集合中指定区间分数的成员数   |
|	**zincrby** key increment member   | 有序集合中对指定成员的分数加上增量 increment   |
|	**zinterstore** destination numkeys key [key ...]   | 计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 key 中   |
|	**zlexcount** key min max   | 在有序集合中计算指定字典区间内成员数量   |
|	**zrange** key start stop [WITHSCORES]   | 通过索引区间返回有序集合成指定区间内的成员   |
|	**zrangebylex** key min max [LIMIT offset count]   | 通过字典区间返回有序集合的成员   |
|	**zrangebyscore** key min max [WITHSCORES] [LIMIT]   | 通过分数返回有序集合指定区间内的成员   |
|	**zrank** key member   | 返回有序集合中指定成员的索引   |
|	**zrem** key member [member ...]   | 移除有序集合中的一个或多个成员   |
|	**zremrangebylex** key min max   | 移除有序集合中给定的字典区间的所有成员   |
|	**zremrangebyrank** key start stop   | 移除有序集合中给定的排名区间的所有成员   |
|	**zremrangebyscore** key min max   | 移除有序集合中给定的分数区间的所有成员   |
|	**zrevrange** key start stop [WITHSCORES]   | 返回有序集中指定区间内的成员，通过索引，分数从高到底   |
|	**zrevrangebyscore** key max min [WITHSCORES]   | 返回有序集中指定分数区间内的成员，分数从高到低排序   | 
|	**zrevrank** key member   | 返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序   |
|	**zscore** key member   | 返回有序集中，成员的分数值   |
|   **zunionstore** destination numkeys key [key ...]   | 计算给定的一个或多个有序集的并集，并存储在新的 key 中   |
| 	**zscan** key cursor [MATCH pattern] [COUNT count]   | 迭代有序集合中的元素（包括元素成员和元素分值）   |


更多内容可参考[redis文档](http://doc.redisfans.com)
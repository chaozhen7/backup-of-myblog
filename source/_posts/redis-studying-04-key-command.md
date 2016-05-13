layout: post
title: redis 学习（四）： 键相关的命令
date: 2016-5-12 12:20:22
tags: 
	- Redis
	- 数据库
comments: true
---

redis 的键命令用于管理 redis 的键。

语法如下：
```
command key_name
```
例如：
```
set ch chenhao
del ch
```

<!--more-->

在以上实例中 `del` 是一个命令， ch 是一个键。 如果键被删除成功，命令执行后输出 (integer) 1，否则将输出 (integer) 0。

redis key 命令：

| 命令	 | 描述 |
|:---:|---|
| DEL key | 该命令用于在 key 存在时删除 key。|
| DUMP key  | 序列化给定 key ，并返回被序列化的值。|
| EXISTS key  | 检查给定 key 是否存在。  |
| EXPIRE key seconds  |为给定 key 设置过期时间。  |
| EXPIREAT key timestamp  | EXPIREAT 的作用和 EXPIRE 类似，都用于为 key 设置过期时间。 不同在于 EXPIREAT 命令接受的时间参数是 UNIX 时间戳(unix timestamp)。  |
| PEXPIRE key milliseconds  |设置 key 的过期时间亿以毫秒计。
| PEXPIREAT key milliseconds-timestamp  | 设置 key 过期时间的时间戳(unix timestamp) 以毫秒计  |
| KEYS pattern  | 查找所有符合给定模式( pattern)的 key 。  |
| MOVE key db  | 将当前数据库的 key 移动到给定的数据库 db 当中。  |
| PERSIST key  | 移除 key 的过期时间，key 将持久保持。  |
| PTTL key  | 以毫秒为单位返回 key 的剩余的过期时间。  |
| TTL key  | 以秒为单位，返回给定 key 的剩余生存时间(TTL, time to live)。  |
| RANDOMKEY  | 从当前数据库中随机返回一个 key 。  |
| 	RENAME key newkey | 修改 key 的名称  |
| RENAMENX key newkey| 仅当 newkey 不存在时，将 key 改名为 newkey | 。
| TYPE key | 返回 key 所储存的值的类型。|


更多命令可以参考 redis 官方文档。
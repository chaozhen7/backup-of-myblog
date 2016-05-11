layout: post
title: Memcached 学习（三）：查找命令介绍
date: 2016-5-11 15:55:22
tags:
	- Memcached
comments: true
copyright: true
---

## 1. get 命令

Memcached get 命令获取存储在 key(键) 中的 value(数据值) ，如果 key 不存在，则返回空。

语法：
```
get key
```
多个 key 使用空格隔开，如下:
```
get key1 key2 key3
```

<!--more-->

参数说明：
**- key：**键值 key-value 结构中的 key，用于查找缓存值。

返回结果：如果找到了就返回对应的key值及相关的信息，如果没有找到就返回空。

## 2. gets 命令

Memcached gets 命令获取带有 CAS 令牌存 的 value(数据值) ，如果 key 不存在，则返回空。

gets 命令的功能类似于基本的 get 命令。两个命令之间的差异在于，gets 返回的信息稍微多一些：64 位的整型值非常像名称/值对的 "版本" 标识符。

语法：
```
gets key
```

多个 key 使用空格隔开，如下:
```
gets key1 key2 key3
```

参数说明：
- **key：** 键值 key-value 结构中的 key，用于查找缓存值。

返回结果：如果找到了就返回对应的key值及相关的信息，如果没有找到就返回空。


## 3. delete 命令

Memcached delete 命令用于删除已存在的 key(键)。
语法：

语法：
```
delete key [noreply]
```

参数说明如下：
- **key：**键值 key-value 结构中的 key，用于查找缓存值。
- **noreply（可选）：** 该参数告知服务器不需要返回数据

命令的输出结果如下：
- **DELETED：**删除成功。
- **ERROR：**语法错误或删除失败。
- **NOT_FOUND：**key 不存在。

## 4. incr 命令

Memcached incr 命令用于对已存在的 key(键) 的数字值进行自增操作。

incr 命令操作的数据必须是十进制的32位无符号整数。

语法：
```
incr key increment_value
```
参数说明：
- key：键值 key-value 结构中的 key，用于查找缓存值。
- increment_value： 增加的数值，不能为负数。

输出结果：
- **NOT_FOUND：** key 不存在。
- **CLIENT_ERROR：** 自增值不是数字。
- **ERROR：** 其他错误，如语法错误等。
如果成功则返回key对应的新的value值。

## 5. decr 命令

Memcached decr 命令用于对已存在的 key(键) 的数字值进行自减操作。与 incr 命令功能正好相反。

命令的语法，参数说明，输出结果均与 incr 命令类似。
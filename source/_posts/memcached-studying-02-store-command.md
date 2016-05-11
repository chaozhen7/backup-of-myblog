layout: post
title: Memcached 学习（二）：存储命令介绍
date: 2016-5-11 13:55:22
tags:
	- Memcached
comments: true
copyright: true
toc: true
---

memcached 中的存储相关的命令主要有： `set`, `add`, `replace`, `append`, `prepend`, `cas`。memcached 中并没有对命令进行分类，这里只是人为的根据命令的功能做了一个分类。

## 1. set 命令

Memcached set 命令用于将 value(数据值) 存储在指定的 key(键) 中。

如果set的key已经存在，该命令可以更新该key所对应的原来的数据，也就是实现更新的作用。

<!--more-->

语法：

```
set key flags exptime bytes [noreply]
value
```

参数说明：
- **key：**键值 key-value 结构中的 key，用于查找缓存值。
- **flags：**可以包括键值对的整型参数，客户机使用它存储关于键值对的额外信息 。
- **exptime：**在缓存中保存键值对的时间长度（以秒为单位，0 表示永远）
- **bytes：**在缓存中存储的字节数
- **noreply（可选）：** 该参数告知服务器不需要返回数据
- **value：**存储的值（始终位于第二行）（可直接理解为key-value结构中的value）

返回结果：
- **STORED：** 成功
- **ERROR：** 失败

实例：
```
set hellokey 0 900 10
hellovalue
```
如果上述输入成功了，那么当我们输入 `get hellokey` 命令时就会得到如下输出
>q VALUE hellokey 0 10
hellovalue
END

## 2. add 命令

Memcached add 命令用于将 value(数据值) 存储在指定的 key(键) 中。

如果 add 的 key 已经存在，则不会更新数据，之前的值将仍然保持相同，并且您将获得响应 NOT_STORED。

语法：
```
add key flags exptime bytes [noreply]
value
```
参数说明： 同 set 命令。

命令输出：
- **NOT_STORED:** add 的key值已经存在（不会更新数据，之前的值将仍然保持相同）。
- **stored:** 成功。


## 3. replace 命令

Memcached replace 命令用于替换已存在的 key(键) 的 value(数据值)。

如果 key 不存在，则替换失败，并且您将获得响应 NOT_STORED。
语法：
```
replace key flags exptime bytes [noreply]
value
```
参数说明：同 set 命令。

输出结果：
- STORED：成功
- NOT_STORED：失败

## 4. append 命令

Memcached append 命令用于向已存在 key(键) 的 value(数据值) 后面追加数据 。

语法：
```
append key flags exptime bytes [noreply]
value
```

参数说明： 同 set 命令。

输出结果：
- **STORED：**保存成功后输出。
- **NOT_STORED：**该键在 Memcached 上不存在。
- **CLIENT_ERROR：**执行错误。

实例：
(1) 首先存储一个键 ch，值为 chenhao
```
set ch 0 900 7
chenhao
```
(2) 获取存储的值
```
get ch
```
> VALUE ch 0 7
chenhao
END

(3) append 命令追加 "memcached"
```
append ch 0 900 9
memcached
```
(4) 获取新的值
```
get ch
```

> VALUE ch 0 16
chenhaomemcached
END

## 5. prepend 命令

Memcached prepend 命令用于向已存在 key(键) 的 value(数据值) 前面追加数据 。

语法：
```
prepend key flags exptime bytes [noreply]
value
```
参数说明： 同 set 命令。

输出结果：
- **STORED：**保存成功后输出。
- **NOT_STORED：**该键在 Memcached 上不存在。
- **CLIENT_ERROR：**执行错误。

## 6. CAS 命令

Memcached CAS（Check-And-Set 或 Compare-And-Swap） 命令用于执行一个"检查并设置"的操作它仅在当前客户端最后一次取值后，该key 对应的值没有被其他客户端修改的情况下， 才能够将值写入。检查是通过cas_token参数进行的， 这个参数是Memcach指定给已经存在的元素的一个唯一的64位值。

语法：
```
cas key flags exptime bytes unique_cas_token [noreply]
value
```
参数说明：
- **key：**键值 key-value 结构中的 key，用于查找缓存值。
- **flags：**可以包括键值对的整型参数，客户机使用它存储关于键值对的额外信息 。
- **exptime：**在缓存中保存键值对的时间长度（以秒为单位，0 表示永远）。
- **bytes：**在缓存中存储的字节数。
- **unique_cas_token：**通过 gets 命令获取的一个唯一的64位值。
- **noreply（可选）： **该参数告知服务器不需要返回数据。
- **value：**存储的值（始终位于第二行）（可直接理解为key-value结构中的value）。

输出结果：
- **STORED：**保存成功后输出。
- **ERROR：**保存出错或语法错误。
- **EXISTS：**在最后一次取值后另外一个用户也在更新该数据。
- **NOT_FOUND：**Memcached 服务上不存在该键值。

实例：

如果没有设置唯一令牌，则 CAS 命令执行错误，如果键 key 不存在，也会执行失败。
(1) 添加键值对
```
set ch 0 900 7
chenhao
```
(2) 通过 gets 命令获取唯一令牌
```
gets ch
```

> VALUE ch 0 7 15
chenhao
END

(3) 使用 cas 命令更新数据
```
cas ch 0 900 4 15
chen
```

(4) 使用 get 命令查看数据是否更新
```
gets ch
```

> VALUE ch 0 4 16
chen
END







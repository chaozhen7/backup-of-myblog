layout: post
title: redis 学习（二）：redis 配置
date: 2016-5-12 10:20:22
tags: 
	- Redis
	- 数据库
comments: true
toc: true
---


在 Redis 有配置文件(redis.conf)可在 Redis 的根目录下找到。可以通过 Redis 的 `config` 命令设置所有 Redis 的配置。

## 1. 查看配置信息

config 命令的语法:
```
config get config_setting_name
```

<!--more-->

参数说明：
- **config_setting_name:** 表示需要查看的参数项名称

例如如果我们输入 `config get loglevel` 可以得到如果结果：
> 1) "loglevel"
> 2) "notice"

如果要查看所有的配置可以用 \* 代替 config_set_name。比如输入 `config get *`，可以得到如下内容（仅列出部分结果）：
>1) "dbfilename"
2) "dump.rdb"
3) "requirepass"
4) ""
5) "masterauth"
6) ""
...
此处省略好多项
...
113) "notify-keyspace-events"
114) ""
115) "bind"
116) ""

## 2. 修改配置信息

可以通过修改 redis.conf 文件或使用 config set 命令来修改配置。

语法：

```
config set config_setting_name config_value
```
参数说明：
- **config_setting_name:** 要设置的配置项名称
- **config_value:** 配置项的值

例如：
```
config set loglevel "notice"
```


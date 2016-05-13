layout: post
title: redis 学习（一）：redis 介绍
date: 2016-5-12 9:20:22
tags: 
	- Redis
	- 数据库
comments: true
toc: true
---

## 1. redis 介绍

REmote DIctionary Server(Redis) 是一个由Salvatore Sanfilippo写的key-value存储系统。

Redis是一个开源的使用ANSI C语言编写、遵守BSD协议、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。

它通常被称为数据结构服务器，因为值（value）可以是 字符串(String), 哈希(Map), 列表(list), 集合(sets) 和 有序集合(sorted sets)等类型。

<!--more-->

Redis 与其他 key - value 缓存产品有以下三个特点：
- Redis支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用。
- Redis不仅仅支持简单的key-value类型的数据，同时还提供list，set，zset，hash等数据结构的存储。
- Redis支持数据的备份，即master-slave模式的数据备份。

Redis 优势:
- 性能极高: Redis能读的速度是110000次/s,写的速度是81000次/s 。
- 丰富的数据类型: Redis支持二进制案例的 Strings, Lists, Hashes, Sets 及 Ordered Sets 数据类型操作。
- 原子: Redis的所有操作都是原子性的，同时Redis还支持对几个操作全并后的原子性执行。
- 丰富的特性: Redis还支持 publish/subscribe, 通知, key 过期等等特性。

## 2. redis 安装

linux 环境
```shell
wget http://download.redis.io/releases/redis-3.2.0.tar.gz
tar xzf redis-3.2.0.tar.gz
cd redis-3.2.0
make
```
make完后 redis-3.2.0 目录下会出现编译后的 redis 服务程序 redis-server,还有用于测试的客户端程序 redis-cli,两个程序位于安装目录 src 目录下：

下面启动redis服务.
```shell
cd src
./redis-server
```

注意这种方式启动redis 使用的是默认配置。也可以通过启动参数告诉redis使用指定配置文件使用下面命令启动。
```shell
cd src
./redis-server redis.conf
```
redis.conf是一个默认的配置文件。我们可以根据需要使用自己的配置文件。

启动redis服务进程后，就可以使用测试客户端程序redis-cli和redis服务交互了。 比如：
```
cd src
./redis-cli
```

Ubuntu 环境下
```
sudo apt-get update
sudo apt-get install redis-server
```
启动
```
redis-server
```
```
redis-cli
```
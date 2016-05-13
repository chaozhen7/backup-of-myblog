layout: post
title: Memcached 学习（一）：介绍和安装
date: 2016-5-11 12:55:22
tags:
	- Memcached
	- 数据库
comments: true
toc: true
---

## 1. Memcached 介绍

Memcached 是一个高性能的分布式内存对象缓存系统，用于动态 Web 应用以减轻数据库负载。它通过在内存中缓存数据和对象来减少读取数据库的次数，从而提高动态、数据库驱动网站的速度。Memcached基于一个存储 键/值 对的hashmap。其守护进程（daemon）是用C写的，但是客户端可以用任何语言来编写，并通过 memcached 协议与守护进程通信。

<!--more-->

本质上，它是一个简洁的 key-value 存储系统。

<img width="400px" src="http://i2.buimg.com/987d9fb3f2156885.png">

## 2. Memcached 安装

Memcached 支持许多平台：Linux、FreeBSD、Solaris、Mac OS，也可以安装在Windows上。

Linux系统安装memcached，首先要先安装libevent库。

```shell
sudo apt-get install libevent libevent-deve     #自动下载安装（Ubuntu/Debian）

yum install libevent libevent-deve              #自动下载安装（Redhat/Fedora/Centos）
```

**Ubuntu/Debian**

```shell
sudo apt-get install memcached
```

**Redhat/Fedora/Centos**

```shell
yum install memcached
```

**FreeBSD**
```shell
portmaster databases/memcached
```
**源代码安装**

从其 [官方网站](http://memcached.org) 下载memcached最新版本。
```shell
wget http://memcached.org/latest                    # 下载最新版本
tar -zxvf memcached-1.x.x.tar.gz                    # 解压源码
cd memcached-1.x.x                                  # 进入目录
./configure --prefix=/usr/local/memcached           # 配置
make && make test                                   # 编译
sudo make install                                   # 安装
```

## 3. Memcached 运行

Memcached命令的运行：

```shell
memcached -h      #命令帮助
```

注意：如果使用自动安装 memcached 命令位于 /usr/local/bin/memcached。

**启动选项：**
-d 是启动一个守护进程；
-m 是分配给Memcache使用的内存数量，单位是MB；
-u 是运行Memcache的用户；
-l 是监听的服务器IP地址，可以有多个地址；
-p 是设置Memcache监听的端口，，最好是1024以上的端口；
-c 是最大运行的并发连接数，默认是1024；
-P 是设置保存Memcache的pid文件。

**作为前台程序运行**

```shell
memcached -p 11211 -m 64m -vv
```

**作为后台服务程序运行**
```shell
memcached -p 11211 -m 64m -d
```

## 4. Memcached 连接

通过上面的方式启动服务器后，可以使用下面的命令连接服务器：

```shell
telnet HOST PORT
```
例如输入：
```
telnet 127.0.0.1  11211
```

输出结果如下：
> Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.



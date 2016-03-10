layout: post
title: ubuntu下安装disksim及ssdmodel模块扩展
date: 2015-9-9 23:23:22
tags: 
	- SSD
	- Disksim
	- Ubuntu
comments: true  
---
## 1. 安装须知
- disksim 目前还没有64位的，所以必须要在32位操作系统下安装
- linux如没安装flex、bison的话，先要安装。
``` shell
sudo apt-get install bison flex
```
- 下载源码安装包
disksim 4.0: [http://www.pdl.cmu.edu/DiskSim/](http://www.pdl.cmu.edu/DiskSim/)
SSD extension:[ http://research.microsoft.com/en-us/downloads/b41019e2-1d2b-44d8-b512-ba35ab814cd4/]( http://research.microsoft.com/en-us/downloads/b41019e2-1d2b-44d8-b512-ba35ab814cd4/)
<!--more-->
## 2.安装步骤
### Step 1
解压
``` shell
$ tar xfz disksim-4.0-with-dixtrac.tar.gz
$ cd disksim-4.0
$ unzip ../ssd-add-on.zip
```
### Step 2
``` shell
$ patch -p1 < ssdmodel/ssd-patch
```
### Step 3
(1) 修改memsmodel/Makefile
```
mems_seektest: mems_seektest.o libmems_internals.a
$(CC) -o $@ mems_seektest.o $(LDFLAGS) $(CFLAGS) -lmems_internals
```
将$(LDFLAGS)放到最后；

(2).修改dixtrac/Makefile
```
LDFLAGS  = -L. -lm -l$(LIBNAME) -ldxtools \
        $(LIBDISKSIM_LDFLAGS) \
        $(MEMSMODEL_LDFLAGS) \
        $(DISKMODEL_LDFLAGS) \
        $(LIBPARAM_LDFLAGS) \
        $(LIBDDBG_LDFLAGS) \
        $(ST_LDFLAGS)
```
将-lm放到最后；

(3).修改src/Makefile
```
LDFLAGS = -lm -L. -ldisksim $(DISKMODEL_LDFLAGS) $(MEMSMODEL_LDFLAGS) \
                            $(LIBPARAM_LDFLAGS) $(LIBDDBG_LDFLAGS)
```
将-lm放到最后；

### Step 4 
将下面的添加到  dixtrac/.paths
```
# path to ssdmodel
export SSDMODEL_PREFIX=../ssdmodel
export SSDMODEL_INCL=$(SSDMODEL_PREFIX)/include
export SSDMODEL_CFLAGS=-I$(SSDMODEL_INCL)
export SSDMODEL_LDPATH=$(SSDMODEL_PREFIX)/lib
export SSDMODEL_LDFLAGS=-L$(SSDMODEL_LDPATH) -lssdmodel
```
修改 dixtrac/Makefile l如下：:
```
$(LIBDISKSIM_LDFLAGS) 
$(MEMSMODEL_LDFLAGS) 
$(DISKMODEL_LDFLAGS) 
$(SSDMODEL_LDFLAGS) 
$(LIBPARAM_LDFLAGS) 
$(LIBDDBG_LDFLAGS) 
$(ST_LDFLAGS)
```
```
CFLAGS = -Wall -g -MD -I. $(DEFINES) -I$(STHREADS) $(DMINCLUDES) 
$(LIBDISKSIM_CFLAGS) 
$(DISKMODEL_CFLAGS) $(LIBPARAM_CFLAGS) $(LIBDDBG_CFLAGS) 
$(SSDMODEL_CFLAGS)
```
### Step 5
编译
``` shell
$ make
```
### Step 6
测试安装是否成功
``` shell
$ cd valid; ./runvalid
$ chmod a+x ../ssdmodel/valid/runvalid
$ cd ../ssdmodel/valid; ./runvalid
```




 
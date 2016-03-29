layout: post
title: disksim with ssdmodel 源码解析（一）：disksim及ssdmodel模块扩展的安装
date: 2015-9-9 23:23:22
tags: 
	- SSD
	- Disksim
	- Ubuntu
comments: true
copyright: true
---
## 1. 安装须知
- disksim 目前还没有64位的，所以必须要在32位操作系统下安装。如果想装在64位系统上请[戳这里](../../../../2016/03/23/disksim-with-ssdmodel-source-analysis-013-use-it-on-64bit/)
- linux如没安装flex、bison的话，先要安装。
``` shell
sudo apt-get install bison flex
```
- 下载源码安装包
disksim 4.0: [http://www.pdl.cmu.edu/DiskSim/](http://www.pdl.cmu.edu/DiskSim/)
SSD extension:[ http://research.microsoft.com/en-us/downloads/b41019e2-1d2b-44d8-b512-ba35ab814cd4/]( http://research.microsoft.com/en-us/downloads/b41019e2-1d2b-44d8-b512-ba35ab814cd4/)
<!--more-->
## 2.安装步骤

### **Step 1**

解压
``` shell
$ tar xfz disksim-4.0-with-dixtrac.tar.gz
$ cd disksim-4.0
$ unzip ../ssd-add-on.zip
```
### **Step 2**

``` shell
$ patch -p1 < ssdmodel/ssd-patch
```


### **如果不想手动修改的话，可以直接调到 step5。** ###


### **Step 3**

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

### **Step 4 **

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



### **step 5**

如果你已经做了 step3 和 step4，请忽略此步骤。 

上面的修改确实很麻烦，如果你没有进行上面的修改可以去我的github上下载patch文件。链接：[戳这里](https://github.com/cighao/disksim-4.0-with-ssdmodel-patch)

将下载后的文件复制到 disksim-4.0 目录下，然后运行 `patch -p1 < modify-patch` 即可。


### **Step 6**
编译
``` shell
$ make
```


### ** Step 7**  ###

测试安装是否成功
``` shell
$ cd valid; ./runvalid
$ chmod a+x ../ssdmodel/valid/runvalid
$ cd ../ssdmodel/valid; ./runvalid
```



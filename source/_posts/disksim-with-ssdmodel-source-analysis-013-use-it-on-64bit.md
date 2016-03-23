layout: post
title: disksim with ssdmodel 源码解析（十三）：如何兼容64位系统
date: 2016-3-23 21:23:22
tags: 
	- SSD
	- Disksim
comments: true  
---

## 1. 写在前面

linux 如没安装flex、bison的话，先要安装。
``` shell
sudo apt-get install bison flex
```
- 下载源码安装包
disksim 4.0: [http://www.pdl.cmu.edu/DiskSim/](http://www.pdl.cmu.edu/DiskSim/)
SSD extension:[ http://research.microsoft.com/en-us/downloads/b41019e2-1d2b-44d8-b512-ba35ab814cd4/]( http://research.microsoft.com/en-us/downloads/b41019e2-1d2b-44d8-b512-ba35ab814cd4/)
<!--more-->

## 2.解压


``` shell
$ tar xfz disksim-4.0-with-dixtrac.tar.gz
$ cd disksim-4.0
$ unzip ../ssd-add-on.zip
```

<!--more-->

## 3. 补丁 ##

可以在我的 github 上面下载 `modify-patch` 和 `64bit-patch` 文件，[戳这里](https://github.com/cighao/disksim-4.0-with-ssdmodel-patch)，[再戳这里](https://github.com/cighao/disksim-4.0-with-ssdmodel-64bit-patch)。下载好后将 `modify-patch` 和 `64bit-patch` 这两个文件都放到 `disksim-4.0` 路径下。

step1. 集成 ssdmodel 
```shell
patch -p1 < ssdmodel/ssd-patch
```

step2. 修改文件

```shell
patch -p1 < modify-patch
```

如果只想在 32 位系统上运行的话，step3可以跳过，直接 make 就行了。

step3. 兼容64位的补丁

```shell
patch -p1 < 64bit-patch
```

## make ##

Just make it! Good luck to you!
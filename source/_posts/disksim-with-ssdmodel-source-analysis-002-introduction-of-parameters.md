layout: post
title: disksim with ssdmodel 源码解析（二）：输入参数的介绍
date: 2016-3-06 10:20:22
tags: 
	- Disksim
	- SSD
comments: true
---

disksim 的参数以文件的方式输入，可在valid文件夹中找到示例，如果安装了 ssdmodel 模块在 ssdmodel/valid 文件夹里也有对应的参数文件示例。根据 disksim 官方手册介绍，参数配置文件主要包括以下三个部分：
- **blocks delimited by{}, 块的定义**
- **instantiations, 实例化**
- **topology specifications, 拓扑结构说明**

<!--more-->

参数文件的组织是相当灵活的，组件不能在定以前引用。每个参数文件必须包含 `Global` 和 `States` 块，使用合成跟踪发生器仿真还必须定义 `proc` 和 `synthio` 块(`disksim_pf { }`：Process-Flow Parameters 合成负载的一些参数, `disksim_synthio { }`：SyntheticWorkloads 合成负载)。

在一个典型的配置中，接着还会再定义一定的总线(bus)、控制器(controller)、和输入输出驱动(iodriver)，然后定义或引用一些存储设备的说明，之后实例化，最后模拟存储仿真的拓扑结构规范定义这些组件之间的互连。

磁盘阵列的数据组织是在 `logorg` 块中描述的。每一个请求都必须属于一个 `logorg`，所以至少有一个 `logorg` 块被定义。

设备的 `Rotational syncrhonization` 可以选择性的在 `syncset` 块中定义。


`adjusting the time scale` 和 `remapping requests from a trace` 可以在 `iosim` 块中定义。


下面以 `ssdmodel/valid/ssd-sr250k.parv` 这个参数文件为例来介绍下各个参数的含义：

其中有些参数是必须设置的（required），有些参数则是可选的（optional）

```python
# Global block
disksim_global Global {
   # 每次模拟开始时会由randomnumber generator产生一个initial seed，决定系统configuration。
   # 若希望实验可还原，可使用相同的InitSeed
   Init Seed = 42, 
   # 决定系统的workload的初始随机数，
   # 当保持Init Seed不变而改变Real Seed时，可得到相同系统configuration下不同workload的实验结果  
   Real Seed = 42,   
   # Statistic warm-up period = 0.0 seconds,
   # 挂载配置文件，statdefs为一个文件名，该文件内定义了收集哪些数据，收集时的bin的大小
   Stat definition file = statdefs
}
```

```python
# iosim block
# 输入 trace 处理的几种方法可以在 iosim block 中配置
disksim_iosim Sim {
   # This specifies a value by which each arrival time in a trace is multiplied.
   I/O Trace Time Scale = 1.0  
}
```

```python
# Stats Block
# 包含五个子块:iodriver、bus、ctlr、device、process flow，参数全由0或1表示
# 1和0分别表示这些参数是否需要输出到输出文件中
disksim_stats Stats {
   iodriver stats = disksim_iodriver_stats {
      Print driver size stats = 1,
      Print driver locality stats = 0,
      Print driver blocking stats = 0,
      Print driver interference stats = 0,
      Print driver queue stats = 1,
      Print driver crit stats = 0,
      Print driver idle stats = 1,
      Print driver intarr stats = 0,
      Print driver streak stats = 1,
      Print driver stamp stats = 1,
      Print driver per-device stats = 0 },
   bus stats = disksim_bus_stats {
      Print bus idle stats = 1,
      Print bus arbwait stats = 1 },
   ctlr stats = disksim_ctlr_stats {
      Print controller cache stats = 1,
      Print controller size stats = 1,
      Print controller locality stats = 1,
      Print controller blocking stats = 1,
      Print controller interference stats = 1,
      Print controller queue stats = 1,
      Print controller crit stats = 1,
      Print controller idle stats = 1,
      Print controller intarr stats = 0,
      Print controller streak stats = 1,
      Print controller stamp stats = 1,
      Print controller per-device stats = 1 },
   device stats = disksim_device_stats {
      Print device queue stats = 1,
      Print device crit stats = 0,
      Print device idle stats = 0,
      Print device intarr stats = 0,
      Print device size stats = 0,
      Print device seek stats = 1,
      Print device latency stats = 1,
      Print device xfer stats = 1,
      Print device acctime stats = 1,
      Print device interfere stats = 0,
      Print device buffer stats = 1 },
   process flow stats = disksim_pf_stats {
      Print per-process stats =  0,
      Print per-CPU stats =  0,
      Print all interrupt stats =  0,
      Print sleep stats =  0}
} # end of stats block

```




下面进入I/OSubsystem Component Specifications，进行具体设备的参数设定。 

包含4块：`device drivers`，`buses`，`controllers`，`storage devices`。 `storage devices` 一般还包含三个子类型：`a disk dodel`, `a simplified`, `fixed-access-time diskmodel`(下文中称 `simpledisk`)

```python
#Device Driver
disksim_iodriver DRIVER0 {
   # 这个参数主要是出于程序的可拓展的目而设定的
   type = 1,
   # 决定每个request的访问时间和队列时间是否一致，还是由下面disk根据自身情况自行计算
   Constant access time = 0.0,  

   # Queue/Scheduler Subcomponents，这是一个 ioqueue.
   Scheduler = disksim_ioqueue {
      # Scheduling算法，如何选择下一个将要处理的request，1代表“先来先服务”，……
      Scheduling policy = 3,  
      # schedule可感知的request的粒度，能否感知到柱面（磁盘每个盘面上有多少磁道）、sector扇区等
      Cylinder mapping strategy = 1,
      # 写延迟
      Write initiation delay = 0.0,
      # 读延迟
      Read initiation delay = 0.0,
      # 对连续读写是否调度到一起执行，布尔类型
      Sequential stream scheme = 0, 
      # 出于优化，可能将多个request调度到一起连续执行，但调度到一起的request读写总和最大不可超过此上限
      Maximum concat size = 128,
      # 如何对待 Overlapping request
      Overlapping request scheme = 0,	
      # 一个sequential stream中允许的最大间隔，超出此间隔的两个request不能在算作同一个stream中
      Sequential stream diff maximum = 0,
      # Muti-queue 超时后的实施方案(对等待时间很长是否调度到更高优先级队列)  
      Scheduling timeout scheme = 0,
      # 过多的排队延迟造成的超时或 time/age factor
      Timeout time/weight = 6,
      # 对timeout queue中的request的调度算法(如何选择下一个执行的request)
      Timeout scheduling = 4,	
      # 标识为高优先级的request是否自动调度到highest-priority queue
      Scheduling priority scheme = 0,
      # 对 priority queue 的调度算法(如何从 priority queue中选择下一个执行的request)
      Priority scheduling = 4
   }, # end of Scheduler
   # 设备驱动是否在任何时候都允许两个或者更多的请求存在存储子系统中
   Use queueing in subsystem = 1
} # end of DRV0 spec
```

```python
# Buses
disksim_bus BUSTOP {
   # bus的类型，1：独占式总线，2：共享式总线
   type = 1,
   # 独占式总线的仲裁类型，1：slot-based priority(SCSI)，2：先来先服务(FCFS)
   Arbitration type = 1,
   # 总线仲裁的时间开销
   Arbitration time = 0.0,
   # 向device driver或host发送一个 512bit 数据块所需的时间(读传输时间ms)
   Read block transfer time = 0.0,
   # 向disk driver发送一个 512bit 数据块所需的时间(写传输时间ms)
   Write block transfer time = 0.0,
   # 是否打印收集的数据
   Print stats =  1
} # end of BUSTOP spec

disksim_bus BUSHBA {
   type = 2,
   Arbitration type = 1,
   Arbitration time = 0.001,

   # PCI-E, with 8 lanes with 8b/10b encoding gives 2.0 Gbps per 
   # lane and with 8 lanes we get about 2.0 GBps. So, bulk sector 
   # transfer time is about 0.238 us. SATA/300 can transfer data 
   # at 300 MBps, which amounts to about 1.6276 us per byte.

   Read block transfer time = 0.0002384,
   Write block transfer time = 0.0002384,
   #Read block transfer time = 0.0016276,
   #Write block transfer time = 0.0016276,

   Print stats =  1
} # end of BUSHBA spec
```

```python
# Controller
disksim_ctlr CTLR0 {
   type = 1, # 控制器的类型   
   Scale for delays = 0.0,  # 控制器所导致的各种延迟的乘法因子
   Bulk sector transfer time = 0.0, # 控制器传输一个 512-bit 数据块所需的时间(ms)
   Maximum queue length = 100,  # 控制器内请求队列的最大长度
   Print stats =  1   # 是否输出控制器相关的数据
} # end of CTLR0 spec
```

下面是关于SSD相关参数的设置

```python
# don't change the order of the following parameters.
# we use Flash chip elements and Elements per gang to
# find number of gang -- we need this info before initializing
# the queue (disksim_ioqueue)
ssdmodel_ssd SSD {
     # vp - this is a percentage of total pages in the ssd
     Reserve pages percentage = 15,  # 预留空间百分比

     # vp - min percentage of free blocks needed. if the free 
     # blocks drop below this, cleaning kicks in
     Minimum free blocks percentage = 5,  # 最少空白块比例，否则触发gc

     # vp - a simple read-modify-erase-write policy = 1 (no longer supported)
     # vp - osr write policy = 2
     Write policy = 2,    # 写策略

     # vp - random = 1 (not supp), greedy = 2, wear-aware = 3
     Cleaning policy = 2,  # gc策略

     # vp - number of planes in each flash package (element)
     Planes per package = 8,  # 每个element包含的plane的数量

     # vp - number of flash blocks in each plane
     Blocks per plane = 2048, # 每个plane包含的block的数量

     # vp - how the blocks within an element are mapped on a plane
     # simple concatenation = 1, plane-pair stripping = 2 (not tested),
     # full stripping = 3
     Plane block mapping = 3,   # element中的块如何映射到每个plane

     # vp - copy-back enabled (1) or not (0)
     Copy back = 1,   # 是否备份数据

     # how many parallel units are there?
     # entire elem = 1, two dies = 2, four plane-pairs = 4
     Number of parallel units = 1,  # 并行单元的数量

     # vp - we use diff allocation logic: chip/plane
     # each gang = 0, each elem = 1, each plane = 2
     Allocation pool logic = 1,

     # elements are grouped into a gang
     Elements per gang = 1,  # 每个gang包含的element的数量

     # shared bus (1) or shared control (2) gang
     Gang share = 1,  # 

     # when do we want to do the cleaning?
     Cleaning in background = 0,   # 什么时候执行gc

     Command overhead =  0.00,   # 指令执行时间
     Bus transaction latency =  0.0,   # 总线处理延迟 

#    Assuming PCI-E, with 8 lanes with 8b/10b encoding.
#    This gives 2.0 Gbps per lane and with 8 lanes we get about
#    2.0 GBps. So, bulk sector transfer time is about 0.238 us.
#    Use the "Read block transfer time" and "Write block transfer time"
#    from disksim_bus above.
     Bulk sector transfer time =  0,   # 同上面(控制器传输一个 512-bit 数据块所需的时间(ms))

     Flash chip elements = 8,   # chip 数量

     Page size = 8,   # page大小

     Pages per block = 64,  # 每个block的page数量

     # vp - changing the no of blocks from 16184 to 16384
     Blocks per element = 16384,

     Element stride pages = 1,

     Never disconnect =  1,
     Print stats =  1,
     Max queue length =  20,
     Scheduler = disksim_ioqueue {
        Scheduling policy =  1,    # 队列的调度算法，如何选取下一个将要响应的request
        Cylinder mapping strategy =  0,  # schedule可感知的request的粒度
        Write initiation delay =  0,    # 写延迟
        Read initiation delay =  0.0,  # 读延迟
        Sequential stream scheme =  0,  # 对连续读写是否调度到一起执行，布尔类型
        Maximum concat size =  0,  # 出于优化，可能将多个request调度到一起连续执行，但调度到一起的request读写总和最大不可超过此上限
        Overlapping request scheme =  0,   # 如何对待 Overlapping request
        # 一个sequential stream中允许的最大间隔，超出此间隔的两个request不能在算作同一个stream中
        Sequential stream diff maximum =  0,
		# Muti-queue 超时后的实施方案(对等待时间很长是否调度到更高优先级队列) 
        Scheduling timeout scheme =  0,
		# 过多的排队延迟造成的超时或 time/age factor
        Timeout time/weight =  0,
		# 对timeout queue中的request的调度算法(如何选择下一个执行的request)
        Timeout scheduling =  0,
        # 标识为高优先级的request是否自动调度到highest-priority queue
	    Scheduling priority scheme =  0,
		# 对 priority queue 的调度算法(如何从 priority queue中选择下一个执行的request)
        Priority scheduling =  1
     },
     Timing model = 1,

     # vp changing the Chip xfer latency from per sector to per byte
     Chip xfer latency = 0.000025,

     Page read latency = 0.025,   # 以page为单位的读延迟
     Page write latency = 0.200,  # 以page为单位的写延迟
     Block erase latency = 1.5    # 以块为单位的擦除延迟
}  # end of SSD spec
```

参数设置完毕，开始实例化。

实例化的语法格式： `instantiate <name list> as <instance name>`

例如： `instantiate [ bus0 ] as BUS0` 表示用 `BUSO` 规范创建一个 `bus0` 实例。

```python
# component instantiation
instantiate [ simfoo ]   as   Sim
instantiate [ statfoo ]   as   Stats

# vp - adding another ssd 
instantiate [ ssd0x0 ]   as   SSD

instantiate [ bustop ]   as   BUSTOP
instantiate [ busHBA0 ]   as   BUSHBA
instantiate [ driver0 ]   as   DRIVER0
instantiate [ ctlrHBA0 ]   as   CTLR0
```

系统的拓扑结构

```python
# system topology
topology disksim_iodriver driver0 [
     disksim_bus bustop [

          ############## HBA 0 #############################
          disksim_ctlr ctlrHBA0 [
               disksim_bus busHBA0 [
                    ############## Flash-device Raid Controller ###############
                    ssdmodel_ssd ssd0x0 []

               ]     # end of bus0
          ]        # end of HBA0

          ############ INSERT MORE HBA's ####################################

     ]           # end of bustop
]              # end of driver0 (and system topology)

```


```python
# no syncsets(无RotationalSynchronization of Devices)
# Disk Array Data Organizations(磁盘阵列逻辑数据组织形式)
# 每一种逻辑结构都可以在 logory 块中配置
disksim_logorg org0 { 
   Addressing mode = Array,  # 逻辑数据组织的编址方式（值为: Array(编成一个统一逻辑设备)或Parts ）
   Distribution scheme = Striped,  # 数据分布策略，体现负载均衡能力(值为: Striped, Random, N.B. 或 Ideal)
   Redundancy scheme = Noredun,  # 冗余方案（值为: Noredun, Shadowed, Parity_disk 或 Parity_rotated）
   
   # vp - added more ssd elements
   devices = [ ssd0x0 ],  # 当前逻辑组织中包含的设备名称列表

   Stripe unit  =  128,  # stripe 单元的大小
   Synch writes for safety =  0,  #
   Number of copies =  2,   # 每个数据磁盘的备份数(仅当Redundancy scheme=shadowed时才有效)
   Copy choice on read =  6,  # 哪个副本负责响应(仅当Redundancy scheme=shadowed时才有效)
   RMW vs. reconstruct =  0.5,  #
   Parity stripe unit =  128,  # stripe 单元的大小(仅当Redundancy scheme=Parity_rotated时才有效)
   Parity rotation type =  1,  # parity 在磁盘中旋转的方式(仅当Redundancy scheme=Parity_rotated时才有效)
   Time stamp interval =  0.000000,   # time stamps 之间的间隔
   Time stamp start time =  60000.000000,  # 第一个time stamp的模拟时间(相对于模拟开始的时间)
   Time stamp stop time =  10000000000.000000,  # 最后一个time stamp的模拟时间(相对于模拟开始的时间)
   Time stamp file name =  stamps
} # end of logorg org0 spec
```

下面设置workload，可有多可generator，每个generator每过一段think time后发出一个request，包括time-criticalreques（generator需等上一个request完成再进入下一个think time）


```python
# Process-Flow Parameters
disksim_pf Proc {
   Number of processors =  1,  # 简单系统级模型所用的处理器的数量
   Process-Flow Time Scale =  1.0  # 模拟处理器执行计算所耗时间的乘法比例因子
} # end of process flow spec

disksim_synthio Synthio {
   Number of I/O requests to generate =  250000, # the number of independent, concurrent, request-generating processes.
   Maximum time of trace generated  =  250000.0, # 产生trace的时间
   System call/return with each request =  0,  # 每一个request是否引发上下文的系统call或return
   Think time from call to request =  0.0,  # 系统call事件和磁盘request事件之间的时间间隔(仅当上一个参数为1时有效)
   Think time from request to return =  0.0, # 磁盘request事件和系统call事件之间的时间间隔(仅当上一个参数为1时有效)

   # generators包含几个概率分布参数，如下：
   # uniform(均匀分布): 需要两个额外的float型参数 minimum 和 maximun
   # normal(正态分布): 需要两个额外的float型参数 mean(均值)和 variance(方差)
   # exponential(指数分布): 需要两个额外的float型参数 mean(均值) base(基数)
   # poisson(泊松分布): 需要两个额外的float型参数 mean(均值) base(基数)
   #
   Generators = [
      disksim_synthgen { # generator 0
         # vp - 85% of ((1 MB - 16K) pages)
         #Storage capacity per device  =  877363,
         Storage capacity per device  =  7018904,  # 每个存储设备上可被访问的地址数
         devices = [ org0 ],   # 可被访问的存储设备的集合
         Blocking factor =  8,  # request的粒度，所有request的访问地址必须是其倍数
         Probability of sequential access =  1.0,  # 产生 sequential request 的概率
         Probability of local access =  0.0,  # 产生 local request 的概率
         Probability of read access =  1.0,  # 产生读请求的概率
         Probability of time-critical request =  0.0,  # 产生 time-critical 请求的概率
         Probability of time-limited request =  0.0,  # 产生 time-limited 请求的概率
         Time-limited think times  = [ normal, 30.0, 100.0  ],   # 符合某种随机分布的一个随机变量(类型是list)
         General inter-arrival times  = [ uniform, 0.0, 1.0  ],  # 下一request地址与上一request不连续时的think time
         Sequential inter-arrival times  = [ uniform, 0.0, 1.0  ], # 下一request地址与上一request连续时的think time
         Local inter-arrival times  = [ exponential, 0.0, 0.0  ],  # 下一request地址位于上一request内时的think time
         Local distances  = [ normal, 0.0, 40000.0  ],  # 当生成local request时，下一request距上一request起始地址的距离
         Sizes  = [ exponential, 0.0, 8.0  ]  #request size
      } # end of generator 0
   ] # end of generator list
} # end of synthetic workload spec
```

上面提到了几种不同的 request 解释如下：
- sequential request: 下一request的地址起始位置正好是上一request的地址结束位置
- local request: 下一request的起始地址距离上一request的起始地址很近，这个距离称为 Local distances


<br>
---------------------
参考：http://blog.csdn.net/ivy_zhang_1101081987/article/details/6803263

更多信息请参考 [disksim官方手册](http://www.pdl.cmu.edu/PDL-FTP/DriveChar/CMU-PDL-08-101.pdf)
## 使用 top 命令观测 CPU 性能

```
# top
top - 11:45:08 up 149 days, 17:31,  1 user,  load average: 0.13, 0.11, 0.04
Tasks:  87 total,   1 running,  53 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.2 us,  0.3 sy,  0.0 ni, 99.5 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :   946176 total,   110160 free,   228472 used,   607544 buff/cache
KiB Swap:   969964 total,   969964 free,        0 used.   540300 avail Mem 

PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND

第一行 当前系统时间,系统已经运行时长,当前用户登录系统个数, 1分钟、5分钟、15分钟的负载情况。（就是 uptime返回的）
第二行：系统进程数据，运行数(running)，休眠数（sleeping），stoped状态个数，zombie状态个数
第三行：cpu状态
	us — 用户空间占用CPU的百分比。
    sy — 内核空间占用CPU的百分比。
    ni — 改变过优先级的进程占用CPU的百分比
    id — 空闲CPU百分比
    wa — IO等待占用CPU的百分比
    hi — 硬中断（Hardware IRQ）占用CPU的百分比
    si — 软中断（Software Interrupts）占用CPU的百分比
第四行：内存状态
    total — 物理内存总量
    free — 空闲内存总量
    used — 使用中的内存总量
    buff/cache — 缓存的内存量
第五行：swap交换分区
	total — 交换区总量
	free — 空闲交换区总量
    used — 使用的交换区总量
	avail Mem  — 缓冲的交换区总量
第七行以下：各进程（任务）的状态监控
    PID — 进程id
    USER — 进程所有者
    PR — 进程优先级
    NI — nice值。负值表示高优先级，正值表示低优先级
    VIRT — 进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES
    RES — 进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
    SHR — 共享内存大小，单位kb
    S — 进程状态。D=不可中断的睡眠状态 R=运行 S=睡眠 T=跟踪/停止 Z=僵尸进程
    %CPU — 上次更新到CPU时间占用百分比
    %MEM — 进程使用的物理内存百分比
    TIME+ — 进程使用的CPU时间总计，单位1/100秒
    COMMAND — 进程名称（命令名/命令行）
```

## 使用 uptime 查看 CPU 负载情况

```
# uptime
11:37:59 up 149 days, 17:23,  1 user,  load average: 0.00, 0.01, 0.00

显示系统已经运行了多长时间，它依次显示下列信息：当前时间、系统已经运行了多长时间、有多少登陆用户、系统在过去的1分钟、5分钟和15分钟内的平均负载

平均负载的最佳值是1，这意味着每个进程都可以立即执行不会错过CPU周期。负载的正常值在不同的系统中有着很大的差别。在单处理器的工作站中，1或2都是可以接受的。然而在多处理器的服务器上你可能看到8到10
主要和 CPU 核数相关

能使用uptime来确定是服务器还是网络出了问题。例如如果网络应用程序运行，运行uptime来了解系统负载是否很高。如果负载不高，这个问题很有可能是由于网络引起的而非服务器。
```

## 通过 vmstat 查看 CPU 繁忙程度

```
vmstat
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 109180 138056 469748    0    0     0     2    0    1  0  0 99  0  0
 
# procs（进程）
	r 等待执行的任务数 展示了正在执行和等待cpu资源的任务个数。
		当这个值超过了cpu个数，就会出现cpu瓶颈。
	B 等待IO的进程数量
		阻塞的进程数量
# memory(内存)
	swpd 正在使用虚拟的内存大小，单位k
		 	如果大于0，表示你的机器物理内存不足了，
 			如果不是程序内存泄露的原因，那么你该升级内存了或者把耗内存的任务迁移到其他机器
	free 空闲内存大小
	buff 已用的buff大小，对块设备的读写进行缓冲
			Linux/Unix系统是用来存储，目录里面有什么内容，权限等的缓存
	cache 已用的cache大小，文件系统的cache
			用来记忆我们打开的文件,给文件做缓冲
			Linux/Unix的聪明之处，把空闲的物理内存的一部分拿来做文件和目录的缓存，
			为了提高 程序执行的性能，当程序使用内存时，buffer/cached会很快地被使用
	inact 非活跃内存大小，即被标明可回收的内存，区别于free和active（当使用-a选项时显示）
	active 活跃的内存大小
# swap
	si 每秒从交换区写入内存的大小（单位：kb/s）
 	so 每秒从内存写到交换区的大小
 		如果这个值大于0，表示物理内存不够用或者内存泄露了，要查找耗内存进程解决掉
# IO
	bi 每秒读取的块数（读磁盘） 块设备每秒接收的块数量，单位是block，
		这里的块设备是指系统上所有的磁盘和其他块设备，现在的Linux版本块的大小为1024bytes
	bo 每秒写入的块数（写磁盘） 块设备每秒发送的块数量，单位是block
		我们读取文件，bo就要大于0。bi和bo一般都要接近0，不然就是IO过于频繁，需要调整
# system
	in 每秒中断数，包括时钟中断
		这两个值越大，会看到由内核消耗的cpu时间sy会越多
	cs 每秒上下文切换数
		调用系统函数，就要进行上下文切换，
		线程的切换，也要进程上下文切换，
		这个值要越小越好，太大了，要考虑调低线程或者进程的数目
		上下文切换次数过多表示你的CPU大部分浪费在上下文切换，导致CPU干正经事的时间少了，CPU没有充分利用
# cpu
	us 用户进程执行消耗cpu时间(user time)
		us的值比较高时，说明用户进程消耗的cpu时间多，
		但是如果长期超过50%的使用，那么我们就该考虑优化程序算法或其他措施了
	sy 系统进程消耗cpu时间(system time)
		sy的值过高时，说明系统内核消耗的cpu资源多，这个不是良性的表现，我们应该检查原因。
		这里us + sy的参考值为80%，如果us+sy 大于 80%说明可能存在CPU不足
		如果太高，表示系统调用时间长，例如是IO操作频繁
	id 空闲时间(包括IO等待时间)
		一般来说 us+sy+id=100
	wa 等待IO时间
		wa过高时，说明io等待比较严重，
		这可能是由于磁盘大量随机访问造成的，也有可能是磁盘的带宽出现瓶颈。
	st 来自于一个虚拟机偷取的CPU时间的百分比
```

## 使用iostat分析IO性能

```
# iostat
avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.28    0.00    0.27    0.00    0.00   99.45
Device             tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
vda               0.71         0.22         4.72    2783883   61062024

avg-cpu: 总体cpu使用情况统计信息，对于多核cpu，这里为所有cpu的平均值
	%user：CPU处在用户模式下的时间百分比。
    %nice：CPU处在带NICE值的用户模式下的时间百分比。
    %system：CPU处在系统模式下的时间百分比。
    %iowait：CPU等待输入输出完成时间的百分比。
    	%iowait的值过高，表示硬盘存在I/O瓶颈
    %steal：管理程序维护另一个虚拟处理器时，虚拟CPU的无意识等待时间百分比。
    %idle：CPU空闲时间百分比。
    	%idle值高，表示CPU较空闲
    	果%idle值高但系统响应慢时，有可能是CPU等待分配内存，此时应加大内存容量。
    	%idle值如果持续低于10，那么系统的CPU处理能力相对较低，表明系统中最需要解决的资源是CPU。

作者：jijs
链接：https://www.jianshu.com/p/5fed8be1b6e8
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
	
Device: 各磁盘设备的IO统计信息
	Device: 以sdX形式显示的设备名称
    tps: 每秒进程下发的IO读、写请求数量
    Blk_read/s: 每秒读扇区数量(一扇区为512bytes)
    Blk_wrtn/s: 每秒写扇区数量
    Blk_read: 取样时间间隔内读扇区总数量
    Blk_wrtn: 取样时间间隔内写扇区总数量
    
# iostat -x
avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.28    0.00    0.27    0.00    0.00   99.45

Device            r/s     w/s     rkB/s     wkB/s   rrqm/s   wrqm/s  %rrqm  %wrqm r_await w_await aqu-sz rareq-sz wareq-sz  svctm  %util
vda              0.01    0.70      0.22      4.72     0.00     0.36   0.18  33.87    1.11    0.15   0.00    20.11     6.76   0.01   0.00

Device
    rrqm/s: 每秒对该设备的读请求被合并次数，文件系统会对读取同块(block)的请求进行合并
    wrqm/s: 每秒对该设备的写请求被合并次数
    r/s: 每秒完成的读次数
    w/s: 每秒完成的写次数
    rkB/s: 每秒读数据量(kB为单位)
    wkB/s: 每秒写数据量(kB为单位)
    avgrq-sz:平均每次IO操作的数据量(扇区数为单位)
    avgqu-sz: 平均等待处理的IO请求队列长度
    	如果avgqu-sz比较大，也表示有当量io在等待。
    await: 平均每次IO请求等待时间(包括等待时间和处理时间，毫秒为单位)
    	如果 await 远大于 svctm，说明I/O 队列太长，io响应太慢，则需要进行必要优化。
    svctm: 平均每次IO请求的处理时间(毫秒为单位)
    	如果 svctm 比较接近 await，说明 I/O 几乎没有等待时间；
    %util: 采用周期内用于IO操作的时间比率，即IO队列非空的时间比率
		如果 %util 接近 100%，说明产生的I/O请求太多，I/O系统已经满负荷，该磁盘可能存在瓶颈。
		
```

## 系统监控应用

```
# nmon
 h = This help                         | r = Resources OS & Proc                                             
| c = CPU Util  C = wide view           | l = longer term CPU averages                                       
| m = Memory & Swap    L=Huge           | V = Virtual Memory                                                 
| n = Network                           | N = NFS                                                           
| d = Disk I/O Graphs  D=Stats          | o = Disks %Busy Map                                               
| k = Kernel stats & loadavg            | j = Filesystem Usage J=reduced                                     
| M = MHz by thread & CPU                                                                                   
| t = TopProcess 1=Priority/Nice/State  | u = TopProc with command line                                     
|     ReOrder by: 3=CPU 4=RAM 5=I/O     |     Hit u twice to update                                         
| g = User Defined Disk Groups          | G = with -g switches Disk graphs                                   
|     [start nmon with -g <filename>]   |     to disk groups only                                           
|                                       | b = black & white mode                                             
| Other Controls:                       |                                                                   
| + = double the screen refresh time    | 0 = reset peak marks (">") to zero                                 
| - = half   the screen refresh time    | space refresh screen now                                           
| . = Display only busy disks & CPU     | q = Quit                                                           
```


## JVM查看日志工具

```
# jvm监控工具 Jconsole / JVisualvm
	java -Djava.rmi.server.hostname=192.168.174.128 -Dcom.sun.management.jmxremote.port=12345 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false  -Dcom.sun.management.jmxremote.access.file=/usr/local/java/jdk1.8.0_172/jre/lib/management/jmxremote.access -Dcom.sun.management.jmxremote.password.file=/usr/local/java/jdk1.8.0_172/jre/lib/management/jmxremote.password -jar server-0.0.1-SNAPSHOT.jar

# 垃圾回收日志分析工具
	http://gceasy.io/
	https://github.com/chewiebug/gcviewer/wiki
	nohup java -jar -Xloggc:/root/outp/gc.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps  outp.jar &
```

## 强引用 软引用 弱引用 虚引用 

```
从JDK 1.2版本开始，对象的引用被划分为4种级别，从而使程序能更加灵活地控制对象的生命周期。这4种级别由高到低依次为：强引用、软引用、弱引用和虚引用。

# 强引用(StrongReference)
	1 强引用是使用最普遍的引用。如果一个对象具有强引用，那垃圾回收器绝不会回收它
		Object strongReference = new Object();
	2 当内存空间不足时，Java虚拟机宁愿抛出OutOfMemoryError错误，使程序异常终止，也不会靠随意回收具有强引用的对象来解决内存不足的问题。
	3 如果强引用对象不使用时，需要弱化从而使GC能够回收 
		strongReference = null;
	4 在一个方法的内部有一个强引用，这个引用保存在Java栈中，而真正的引用内容(Object)保存在Java堆中。当这个方法运行完成后，就会退出方法栈，则引用对象的引用数为0，这个对象会被回收。
	5 如果这个strongReference是全局变量时，就需要在不用这个对象时赋值为null，因为强引用不会被垃圾回收。

# 软引用(SoftReference)
	1 一个对象只具有软引用，则内存空间充足时，垃圾回收器就不会回收它；如果内存空间不足了，就会回收这些对象的内存。只要垃圾回收器没有回收它，该对象就可以被程序使用
		String str = new String("abc");
    	SoftReference<String> softReference = SoftReference<String>(str)
    2 软引用可以和一个引用队列(ReferenceQueue)联合使用。如果软引用所引用对象被垃圾回收，JAVA虚拟机就会把这个软引用加入到与之关联的引用队列中
    3 软引用对象是在jvm内存不够的时候才会被回收，我们调用System.gc()方法只是起通知作用，JVM什么时候扫描回收对象是JVM自己的状态决定的。就算扫描到软引用对象也不一定会回收它，只有内存不够的时候才会回收。
    4 当内存不足时，JVM首先将软引用中的对象引用置为null，然后通知垃圾回收器进行回收
    5 垃圾收集线程会在虚拟机抛出OutOfMemoryError之前回收软引用对象，而且虚拟机会尽可能优先回收长时间闲置不用的软引用对象。对那些刚构建的或刚使用过的“较新的”软对象会被虚拟机尽可能保留，这就是引入引用队列ReferenceQueue的原因

# 弱引用(WeakReference)
	弱引用与软引用的区别在于：只具有弱引用的对象拥有更短暂的生命周期。
	在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存
	由于垃圾回收器是一个优先级很低的线程，因此不一定会很快发现那些只具有弱引用的对象。
	如果一个对象是偶尔(很少)的使用，并且希望在使用时随时就能获取到，但又不想影响此对象的垃圾收集，那么你应该用Weak Reference来记住此对象。
	import java.lang.ref.Reference;

# 虚引用(PhantomReference)
	虚引用并不会决定对象的生命周期。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收
	虚引用主要用来跟踪对象被垃圾回收器回收的活动。
	虚引用必须和引用队列(ReferenceQueue)联合使用。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中
		import java.lang.ref.PhantomReference;
        String str = new String("abc");
        ReferenceQueue queue = new ReferenceQueue();
        // 创建虚引用，要求必须与一个引用队列关联
        PhantomReference pr = new PhantomReference(str, queue);
   	程序可以通过判断引用队列中是否已经加入了虚引用，来了解被引用的对象是否将要进行垃圾回收。如果程序发现某个虚引用已经被加入到引用队列，那么就可以在所引用的对象的内存被回收之前采取必要的行动。
   	
引用类型	被垃圾回收时间			用途				生存时间
强引用			从来不会		  对象的一般状态		JVM停止运行时终止
软引用			当内存不足时		 对象缓存			 内存不足时终止
弱引用			正常垃圾回收时		对象缓存			垃圾回收后终止
虚引用			正常垃圾回收时		跟踪对象的垃圾回收	 垃圾回收后终止
```

## Java GC机制

```
# 什么是GC
	JVM 中，程序计数器(Program Counter Register)、虚拟机栈(VM Stack)、本地方法栈(Native Method Stack)都是随线程而生随线程而灭，栈帧随着方法的进入和退出做入栈和出栈操作，实现了自动的内存清理，因此，我们的内存垃圾回收主要集中于 java 堆(Heap)和方法区(Method Area)中，在程序运行期间，这部分内存的分配和使用都是动态的。
	
# GC的对象
	需要进行回收的对象就是已经没有存活的对象，判断一个对象是否存活常用的有两种办法：引用计数和可达分析。
		1 引用计数：每个对象有一个引用计数属性，新增一个引用时计数加1，引用释放时计数减1，计数为0时可以回收。此方法简单，无法解决对象相互循环引用的问题。
		2 可达性分析（Reachability Analysis）：从GC Roots开始向下搜索，搜索所走过的路径称为引用链。当一个对象到GC Roots没有任何引用链相连时，则证明此对象是不可用的。不可达对象。
	
# 什么时候触发GC
	1 程序调用System.gc时可以触发
	2 系统自身来决定GC触发的时机（根据Eden区和From Space区的内存大小来决定。当内存大小不足时，则会启动GC线程并停止应用线程）
	GC又分为 minor GC 和 Full GC (也称为 Major GC )
		Minor GC触发条件：当Eden区满时，触发Minor GC。
		Full GC触发条件：
  			1 调用System.gc时，系统建议执行Full GC，但是不必然执行
  			2 老年代空间不足
 			3 方法去空间不足
 			4 通过Minor GC后进入老年代的平均大小大于老年代的可用内存
			5 由Eden区、From Space区向To Space区复制时，对象大小大于To Space可用内存，则把该对象转存到老年代，且老年代的可用内存小于该对象大小

# GC常用算法
## 标记-清除算法
    为每个对象存储一个标记位，记录对象的状态（活着或是死亡）。
    分为两个阶段:
        1 标记阶段，这个阶段内，为每个对象更新标记位，检查对象是否死亡；
        2 清除阶段，该阶段对死亡的对象进行清除，执行 GC 操作。

    优点:
        标记—清除算法中每个活着的对象的引用只需要找到一个即可，找到一个就可以判断它为活的。					不移动对象的位置。

    缺点:
        效率比较低（递归与全堆对象遍历）。每个活着的对象都要在标记阶段遍历一遍；
        所有对象都要在清除阶段扫描一遍，因此算法复杂度较高。
        没有移动对象，导致可能出现很多碎片空间无法利用的情况。

## 标记-压缩算法（标记-整理）
    标记-清除法的一个改进版。
    在标记阶段，该算法也将所有对象标记为存活和死亡两种状态；
    在第二个阶段，该算法并没有直接对死亡的对象进行清理，
    而是将所有存活的对象整理一下，放到另一处空间，
    然后把剩下的所有对象全部清除。这样就达到了标记-整理的目的。

	优点:
    	该算法不会像标记-清除算法那样产生大量的碎片空间。
    	
	缺点:
		如果存活的对象过多，整理阶段将会执行较多复制操作，导致算法效率降低。
	
## 复制算法
    该算法将内存平均分成两部分，然后每次只使用其中的一部分，
    当这部分内存满的时候，将内存中所有存活的对象复制到另一个内存中，
    然后将之前的内存清空，只使用这部分内存，循环下去。

    优点:
    	实现简单；不产生内存碎片
    缺点:
    	每次运行，总有一半内存是空的，导致可使用的内存空间只有原来的一半

## 分代收集算法
    现在的虚拟机垃圾收集大多采用这种方式，
    它根据对象的生存周期，将堆分为新生代(Young)和老年代(Tenure)。
    在新生代中，由于对象生存期短，每次回收都会有大量对象死去，那么这时就采用复制算法。
    老年代里的对象存活率较高，没有额外的空间进行分配担保，所以可以使用标记-整理 或者 标记-清除。

# 垃圾收集器 (收集算法是内存回收的方法论，垃圾收集器就是内存回收的具体实现)
## Serial收集器
    串行收集器是最古老，最稳定以及效率高的收集器
    可能会产生较长的停顿，只使用一个线程去回收
    -XX:+UseSerialGC
    
## ParNew收集器
    -XX:+UseParNewGC（new代表新生代，所以适用于新生代）
    新生代并行
    老年代串行
    Serial收集器新生代的并行版本
    在新生代回收时使用复制算法
    多线程，需要多核支持
    -XX:ParallelGCThreads 限制线程数量
    
## Parallel收集器
    类似ParNew 
    新生代复制算法 
    老年代标记-压缩 
    更加关注吞吐量 
    -XX:+UseParallelGC  
    	使用Parallel收集器 + 老年代串行
    -XX:+UseParallelOldGC 
    	使用Parallel收集器+ 老年代并行
    	
    -XX:MaxGCPauseMills
    	最大停顿时间，单位毫秒
    	GC尽力保证回收时间不超过设定值
    -XX:GCTimeRatio 
   	 	0-100的取值范围
    	垃圾收集时间占总时间的比
    	默认99，即最大允许1%时间做GC

    这两个参数是矛盾的。因为停顿时间和吞吐量不可能同时调优

## CMS收集器
    Concurrent Mark Sweep 并发标记清除（应用程序线程和GC线程交替执行）
    使用标记-清除算法
    并发阶段会降低吞吐量（停顿时间减少，吞吐量降低）
    老年代收集器（新生代使用ParNew）
    -XX:+UseConcMarkSweepGC
        
	尽可能降低停顿
	会影响系统整体吞吐量和性能

	比如，在用户线程运行过程中，分一半CPU去做GC，系统性能在GC阶段，反应速度就下降一半
	清理不彻底 

	因为在清理阶段，用户线程还在运行，会产生新的垃圾，无法清理
    因为和用户线程一起运行，不能在空间快满时再清理（因为也许在并发GC的期间，用户线程又申请了大量内存，导致内存不够） 

        -XX:CMSInitiatingOccupancyFraction设置触发GC的阈值
        	如果不幸内存预留空间不够，就会引起concurrent mode failure
        	一旦 concurrent mode failure产生，将使用串行收集器作为后备。

        CMS也提供了整理碎片的参数：
            -XX:+ UseCMSCompactAtFullCollection Full GC后，进行一次整理
                整理过程是独占的，会引起停顿时间变长

            -XX:+CMSFullGCsBeforeCompaction  
                设置进行几次Full GC后，进行一次碎片整理

            -XX:ParallelCMSThreads 
                设定CMS的线程数量（一般情况约等于可用CPU数量）
        	
	CMS的提出是想改善GC的停顿时间，在GC过程中的确做到了减少GC时间，但是同样导致产生大量内存碎片，又需要消耗大量时间去整理碎片，从本质上并没有改善时间。 
        
## G1收集器
    G1是目前技术发展的最前沿成果之一，
    HotSpot开发团队赋予它的使命是未来可以替换掉JDK1.5中发布的CMS收集器。
    与CMS收集器相比G1收集器有以下特点：
    	(1) 空间整合，G1收集器采用标记整理算法，不会产生内存空间碎片。分配大对象时不会因为无法找到连续空间而提前触发下一次GC。
        (2)可预测停顿，这是G1的另一大优势，降低停顿时间是G1和CMS的共同关注点，但G1除了追求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为N毫秒的时间片段内，消耗在垃圾收集上的时间不得超过N毫秒，这几乎已经是实时Java（RTSJ）的垃圾收集器的特征了。

	上面提到的垃圾收集器，收集的范围都是整个新生代或者老年代，而G1不再是这样。
	使用G1收集器时，Java堆的内存布局与其他收集器有很大差别，
	它将整个Java堆划分为多个大小相等的独立区域（Region），
	虽然还保留有新生代和老年代的概念，但新生代和老年代不再是物理隔阂了，它们都是一部分（可以不连续）Region的集合。

	G1的新生代收集跟ParNew类似，当新生代占用达到一定比例的时候，开始出发收集。
	和CMS类似，G1收集器收集老年代对象会有短暂停顿。

	步骤：
		(1)标记阶段，
			首先初始标记(Initial-Mark),这个阶段是停顿的(Stop the World Event)，
			并且会触发一次普通Mintor GC。对应GC log:GC pause (young) (inital-mark)

		(2)Root Region Scanning，
			程序运行过程中会回收survivor区(存活到老年代)，这一过程必须在young GC之前完成。

		(3)Concurrent Marking，
			在整个堆中进行并发标记(和应用程序并发执行)，此过程可能被young GC中断。
			在并发标记阶段，若发现区域对象中的所有对象都是垃圾，那个这个区域会被立即回收(图中打X)。
			同时，并发标记过程中，会计算每个区域的对象活性(区域中存活对象的比例)。

		(4)Remark, 再标记，会有短暂停顿(STW)。
			再标记阶段是用来收集 并发标记阶段 产生新的垃圾(并发阶段和应用程序一同运行)；
			G1中采用了比CMS更快的初始快照算法:snapshot-at-the-beginning (SATB)。

		(5)Copy/Clean up，多线程清除失活对象，会有STW。
			G1将回收区域的存活对象拷贝到新区域，清除Remember Sets，
			并发清空回收区域并把它返回到空闲区域链表中。

		(6)复制/清除过程后。回收区域的活性对象已经被集中回收到深蓝色和深绿色区域。
```

## JVM内存区域详解

```
JVM区域总体分两类，heap区和非heap区。
heap区又分为：
    Eden Space（伊甸园）
    Survivor Space(幸存者区)
    Old Gen（老年代）
    
非heap区又分：
    Code Cache(代码缓存区)
    Perm Gen（永久代）
    Jvm Stack(java虚拟机栈)
    Local Method Statck(本地方法栈)

Eden Space:
	字面意思是伊甸园，对象被创建的时候首先放到这个区域，
	进行垃圾回收后，不能被回收的对象被放入到空的survivor区域。
	
Survivor Space幸存者区:
	用于保存在eden space内存区域中经过垃圾回收后没有被回收的对象。
	Survivor有两个，分别为To Survivor、 From Survivor，这个两个区域的空间大小是一样的。
	执行垃圾回收的时候 Eden区域不能被回收的对象被放入到空的survivor（也就是To Survivor，同时Eden区域的内存会在垃圾回收的过程中全部释放），
	另一个survivor（即From Survivor）里不能被回收的对象也会被放入这个survivor（即To Survivor），然后To Survivor 和 From Survivor的标记会互换，始终保证一个survivor是空的。

Old Gen老年代
	用于存放新生代中经过多次垃圾回收仍然存活的对象，也有可能是新生代分配不了内存的大对象会直接进入老年代。经过多次垃圾回收都没有被回收的对象，这些对象的年代已经足够old了，就会放入到老年代。
	
Eden Space和Survivor Space都属于新生代，新生代中执行的垃圾回收被称之为Minor GC（因为是对新生代进行垃圾回收，所以又被称为Young GC），每一次Young GC后留下来的对象age加1。

当老年代被放满的之后，虚拟机会进行垃圾回收，称之为Major GC。由于Major GC除并发GC外均需对整个堆进行扫描和回收，因此又称为Full GC。

heap区即堆内存，整个堆大小=年轻代大小 + 老年代大小。堆内存默认为物理内存的1/64(<1GB)；默认空余堆内存小于40%时，JVM就会增大堆直到-Xmx的最大限制，可以通过MinHeapFreeRatio参数进行调整；默认空余堆内存大于70%时，JVM会减少堆直到-Xms的最小限制，可以通过MaxHeapFreeRatio参数进行调整。

Code Cache代码缓存区:
	它主要用于存放JIT所编译的代码。CodeCache代码缓冲区的大小在client模式下默认最大是32m，在server模式下默认是48m，这个值也是可以设置的，它所对应的JVM参数为ReservedCodeCacheSize 和 InitialCodeCacheSize，可以通过如下的方式来为Java程序设置

	-XX:ReservedCodeCacheSize=128m

	CodeCache缓存区是可能被充满的，
	当CodeCache满时，后台会收到CodeCache is full的警告信息，如下所示：
“CompilerThread0” java.lang.OutOfMemoryError: requested 2854248 bytes for Chunk::new. Out of swap space?

Perm Gen:
	全称是Permanent Generation space，是指内存的永久保存区域，因而称之为永久代。
	这个内存区域用于存放Class和Meta的信息，Class在被 Load的时候被放入这个区域。
	因为Perm里存储的东西永远不会被JVM垃圾回收的，
	所以如果你的应用程序LOAD很多CLASS的话，就很可能出现PermGen space错误。
	默认大小为物理内存的1/64。
```

## AlwaysPreTouch

```
-XX:+AlwaysPreTouch
启动时的时候,把所有内存预先分配
```

## jvisualvm 获取JVM性能数据

```
# 远程配置
java -Xdebug -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false  -Dcom.sun.management.jmxremote.port=10000 -Djava.rmi.server.hostname=192.169.100.159
```

## JMC 获取 Java应用详细性能数据

```
jcmd <pid> JFR.start
jcmd <pid> JFR.dump filename=log.jfr
jcmd <pid> JFR.stop

https://www.oracle.com/java/technologies/jdk-mission-control.html
```




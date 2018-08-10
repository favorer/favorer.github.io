[GC参考](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/toc.html)

[TOC]
# gc度量
There are two primary measures of garbage collection performance:

* Throughput is the percentage of total time not spent in garbage collection considered over long periods of time. Throughput includes time spent in allocation (but tuning for speed of allocation is generally not needed).

* Pauses are the times when an application appears unresponsive because garbage collection is occurring.

# gc调试参数
* -verbose:gc
```
[GC 325407K->83000K(776768K), 0.2300771 secs]
[GC 325816K->83372K(776768K), 0.2454258 secs]
[Full GC 267628K->83769K(776768K), 1.8479984 secs]
```
* -XX:+PrintGCDetails
```
[GC [DefNew: 64575K->959K(64576K), 0.0457646 secs] 196016K->133633K(261184K), 0.0459067 secs]
```
* -XX:+PrintGCTimeStamps
```
111.042: [GC 111.042: [DefNew: 8128K->8128K(8128K), 0.0000505 secs]111.042: [Tenured: 18154K->2311K(24576K), 0.1290354 secs] 26282K->2311K(32704K), 0.1293306 secs]
```
# gc调优参数
* 调整整体和年轻代和永久代heap size
* the features of the adaptive size policy
## 非并行gc（non-parallel collector）
1. 总堆设置
  *  -Xms\<min> -Xmx\<max> 总堆的初始和上限值
  *  -XX:MinHeapFreeRatio=\<minimum> -XX:MaxHeapFreeRatio=<maximum> 堆空闲比例
>As noted in Table 4-1, "Default Parameters for 64-Bit Solaris Operating System", the default maximum heap size is a value that is calculated by the JVM. The calculation used in Java SE for the parallel collector and the server JVM are now used for all the garbage collectors. Part of the calculation is an upper limit on the maximum heap size that is different for 32-bit platforms and 64-bit platforms. See the section Default Heap Size in The Parallel Collector. There is a similar calculation for the client JVM, which results in smaller maximum heap sizes than for the server JVM.
The following are general guidelines regarding heap sizes for server applications:
> * Unless you have problems with pauses, try granting as much memory as possible to the virtual machine. The default size is often too small.
>* Setting -Xms and -Xmx to the same value increases predictability by removing the most important sizing decision from the virtual machine. However, the virtual machine is then unable to compensate if you make a poor choice.
>* In general, increase the memory as you increase the number of processors, since allocation can be parallelized.

>Table 4-1 Default Parameters for 64-Bit Solaris Operating System

Parameter|Default Value
---|---
MinHeapFreeRatio|40
MaxHeapFreeRatio|70
-Xms|6656k
-Xmx|calculated

2. 年轻代设置
*  -XX:NewRatio=3 该值设置年轻代和永久代比为1:3
* NewSize，MaxNewSize 或者通过设置具体年轻代的初始和上限值
* -XX:SurvivorRatio=6 edge和survivor比为6:1。注意有2个survior。survivor和年轻代比为1:8
 -XX:+PrintTenuringDistribution 展示复制到永久代前的存活周期阈值。
> Table 4-2 Default Parameter Values for Survivor Space Sizing

Parameter|	Server JVM Default Value
---|---
NewRatio|2
NewSize|1310M
MaxNewSize|not limited
SurvivorRatio|8

>The following are general guidelines for server applications:
>
>* First decide the maximum heap size you can afford to give the virtual machine. Then plot your performance metric against young generation sizes to find the best setting.
>
>   * Note that the maximum heap size should always be smaller than the amount of memory installed on the machine to avoid excessive page faults and thrashing.
>
>* If the total heap size is fixed, then increasing the young generation size requires reducing the tenured generation size. Keep the tenured generation large enough to hold all the live data used by the application at any given time, plus some amount of slack space (10 to 20% or more).
>
>* Subject to the previously stated constraint on the tenured generation:
>
>   * Grant plenty of memory to the young generation.
>
>   * Increase the young generation size as you increase the number of processors, because allocation can be parallelized.

## GC类型
* serial collector 串行。可用 **-XX:+UseParallelGC** 开启。单线程收集。适用于单核机器，或者多核机器小于100MB内存。
* parallel collector(aka throughput collector) 并行收集器。可用 **-XX:+UseParallelGC** 开启
   * Parallel compaction 并行压缩 默认开启。可以多线程执行major collections回收。随着-XX:+UseParallelGC开启，若关闭使用 **-XX:-UseParallelOldGC**
* concurrent collector(targeted to minimize pauses) CMS或G1收集器。重点减少stw暂停，但会降低吞吐量，影响性能。HotSpot VM提供CMS和G1.分别用 **-XX:+UseConcMarkSweepGC**或 **-XX:+UseG1GC** 开启。

>Selecting a Collector
>Unless your application has rather strict pause time requirements, first run your application and allow the VM to select a collector. If necessary, adjust the heap size to improve performance. If the performance still does not meet your goals, then use the following guidelines as a starting point for selecting a collector.
>
>* If the application has a small data set (up to approximately 100 MB), then select the serial collector with the option -XX:+UseSerialGC.
>
>* If the application will be run on a single processor and there are no pause time requirements, then let the VM select the collector, or select the serial collector with the option -XX:+UseSerialGC.
>
>* If (a) peak application performance is the first priority and (b) there are no pause time requirements or pauses of 1 second or longer are acceptable, then let the VM select the collector, or select the parallel collector with -XX:+UseParallelGC.
>
>* If response time is more important than overall throughput and garbage collection pauses must be kept shorter than approximately 1 second, then select the concurrent collector with -XX:+UseConcMarkSweepGC or -XX:+UseG1GC.
>
>These guidelines provide only a starting point for selecting a collector because performance is dependent on the size of the heap, the amount of live data maintained by the application, and the number and speed of available processors. Pause times are particularly sensitive to these factors, so the threshold of 1 second mentioned previously is only approximate: the parallel collector will experience pause times longer than 1 second on many data size and hardware combinations; conversely, the concurrent collector may not be able to keep pauses shorter than 1 second on some combinations.


## 并行GC(吞吐量优先)
 使用-XX:+UseParallelGC开启。minor和major回收都并行回收。gc日志输出和串行GC基本一样。
 * 默认配置：硬件上有N个线程时，若N大于8，回收线程数为5/8 * N;若N小于8，回收线程为8.特定机器上比例降为5/16.
 * 命令行设置回收线程数：**-XX:ParallelGCThreads=\<N>**
 * 每个回收线程在minor gc时会保留部分永久代空间用于对象晋级老年代，可能会导致碎片效应。减少回收线程和增大永久代空间可以减缓该现象。
### 分代
并行收集器采用更自动化的调优方法，不需要指定底层的分代大小等配置。一般指定三种配置：
 * 最大回收暂停时间： **-XX:MaxGCPauseMillis=\<N>** （N：毫秒）。默认不配置没有最大暂停时间目标。配置后，堆大小和其他gc相关参数会自动调整，尝试将gc暂停时间小于指定时间。这样有可能导致降低gc吞吐量，而且gc时间有可能会超过指定时间。
 * 吞吐量：**-XX:GCTimeRatio=\<N>**。吞吐量的衡量计算式gc时间除以非gc时间（应用耗时）的比例。该设置配置gc耗时占比为1/(1+N)。默认值为99，即gc时间占1%。
   比如-XX:GCTimeRatio=19会设置gc占总耗时的1/20。
 * 最小内存Footprint(占用空间): **-Xmx\<N>** 最大堆空间.gc会在其他目标匹配时，最小化堆空间
### 目标优先级
 按如下顺序优先匹配：最大暂定时间，吞吐量，最小内存。
### 分代大小调整
 每次gc后（显示调用System.gc()除外）会更新平均暂停时间。然后测试相关目标是否匹配，否则做出分代大小的调整。

 大小调整会按固定比例向需要的大小分步调整。增长和收缩的比例是不同的。默认增长比例为20%，收缩比例为5%。可以通过选项控制。 **-XX:YoungGenerationSizeIncrement=\<Y>**, **-XX:TenuredGenerationSizeIncrement=\<T>** 分别指定年轻代和永久代的增长百分比。 **-XX:AdaptiveSizeDecrementScaleFactor=\<D>** 指定收缩的因子，比如对应增长率为X=20%，D为4，则收缩比例为20/4=5%，即X/D %。

 启动阶段，增长比例会有一个附加值。实际增长比例为增长比例+附加比例。附加比例会随着gc次数增长而衰减，对长期没有影响。主要为了提高启动阶段的性能。收缩时没有附加比例。

 如果最大暂停时间目标没达成，一次只会收缩一代。若年轻代和永久代都未达到目标，则优先收缩暂停时间最长的代。

 如果吞吐量目标没达成，两代都会增长。各代会按照各自占总gc时间的比例，乘以前面增长比例，得到实际的增长比例。比如年轻代gc时间占总gc时长的25%，并且指定年轻代增长比例为20%，则实际会增长25% * 20%=5%.

### 堆大小默认值和设置
若命令行选项未指定初始和最大堆，则按照机器内存计算。
* Client JVM
  * JVM最大内存默认为物理内存的一半，当物理内存最大为192MB。或者为物理内存的四分之一，当物理内存有1GB时。
     *  比如机器有128MB内存，jvm最大堆为64MB。机器有1GB或更大时，jvm最大堆为256MB。
  * JVM初始内存最少8MB，或最大1G物理内存的1/64.
  * 年轻代最大内存为最大堆的1/3.
* Server JVM  
  * 和Client类似，不过默认值更大。32位机器默认最大值若有4G或更大内存时可以到1GB。64位机器若机器有128GB或更大时，可以到32GB.
* 指定堆大小
  * 和前面一样通过-Xms和-Xmx来指定初始和最大值。JVM会在堆使用空间和性能间寻找实际堆设置的平衡。
  * 默认值可能受其他参数影响。可以通过 **-XX:+PrintFlagsFinal** 的输出中查找 **MaxHeapSize** 对应的实际最大值。
    比如``` java -XX:+PrintFlagsFinal <GC options> -version | grep MaxHeapSize ```

### OutOfMemoryError
 如果gc耗费了过多时间，并行收集器会抛出OutOfMemoryError。当总时间的98%用于GC，并且只回收了2%的空间。此时就发生OutOfMemoryError。这样是为了防止由于堆设置太小导致应用几乎停滞不操作。可以通过 **-XX:-UseGCOverheadLimit** 禁用该特性。

 ## 并发GC
  Java Hotspot VM在JDK8中使用两种并行GC：CMS和G1。
  * [CMS（Concurrent Mark Sweep Collector）](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/cms.html#concurrent_mark_sweep_cms_collector):倾向更短GC暂停，回收过程共享处理器。
  * [G1（Garbage-First Garbage Collector）](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/g1_gc.html#garbage_first_garbage_collection) ：为服务器多处理器和大内存而生。尽量缩短GC暂停同时，最大可能提高吞吐量。
### 并发成本
  用可能属于应用的处理器资源换取更短的major gc暂停时间。有N个处理器的系统并发回收会使用K个处理器，其中1<=K<=ceiling{N/4} （k可能会变）.并发阶段使用处理器，会导致应用自身的吞吐量比其他垃圾回收器略低。

### 其他材料引用   
 * [The Garbage-First Garbage Collector](http://www.oracle.com/technetwork/java/javase/tech/g1-intro-jsp-135488.html)
 * [Garbage-First Garbage Collector Tuning](http://www.oracle.com/technetwork/articles/java/g1gc-1984535.html)

## CMS回收器
  用 **-XX:+UseConcMarkSweepGC** 开启。

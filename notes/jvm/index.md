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
  用 **-XX:+UseConcMarkSweepGC** 开启。设计倾向于更短的gc暂停，并能在应用运行时共享处理器资源。适用于有大量永久代长期存活对象，且有两个或以上处理器。
  和其他gc一样，cms是分代的，minor和major回收都会发生。cms在major回收阶段会尝试使用多个gc线程在应用线程运行时一起跟踪可达对象。在每个gc周期，cms会在回收刚开始暂停一个很短的时间，然后在回收中期再次暂停。第二次暂停会更久。每次暂停阶段都是多线程回收。剩余回收阶段（包括跟踪存活对象，和清理不可达对象）会使用单个或多个gc线程和应用一起并发执行。minor gc会和正在进行的major gc交替执行，执行方式类似并行gc（应用线程在minor回收时会暂停）。

### 并发模式失败
  cms使用一个或多个线程和应用线程一起运行的目标是在永久代变满前进行垃圾回收。通常情况下，cms在应用运行时，做了大部分的跟踪和清理工作，所以应用才会感受到较短的暂停。然后，如果cms在永久代变满前，无法回收不可达对象，或者分配空间在年老代无法获取指定可用的空闲space block；那么应用将会被暂停，所有应用线程将会停止。这个场景就无法完成并发回收，被称作 **并发模式失败**，通常以为这需要调整cms的回收参数。如果并发回收被显示的gc调用（System.gc()）中断或者被诊断工具提供信息等中断，**并发模式中断** 会被报告。
### gc过度耗时与OutOfMemoryError
  CMS在gc消耗了太多时间时会抛出OutOfMemoryError。一般超过98%的总时间花在gc，但是少于2%的堆空间被回收时，抛出OutOfMemoryError。该特性设计防止由于堆太小导致应用额外运行时间，但是几乎没有进展。如果需要，可以使用 **-XX:-UseGCOverheadLimit** 禁用该特性。
  该策略和其他并发回收器一样，除了时间计算上，并行回收的时间没有算在98%的时间限制内。换句话说，就是过度gc时间只计算应用暂停时的回收时间，没包含和应用线程一起执行的时间。这种回收典型场景是由于并发模式失败或显示gc请求（例如System.gc）导致。

  ### 浮动垃圾
   CMS和HotSpot的其他回收器类似，是跟踪堆内的所有可达对象实现。在Richard Jones和Rafael D. Lins的出版物 **《Garbage Collection: Algorithms for Automated Dynamic Memory》** 中描述的，这是增量更新的回收器。因为应用线程和gc线程在major回收时并发执行时，被跟踪的对象可能会在回收过程刚结束后，变的不可达。这类不可达对象尚未被标记为可回收，被称作浮动垃圾。浮动垃圾的数量依赖于回收阶段的持续时间和引用更新或称作突变的频率。此外，由于年青代和永久代独立回收，各自作为彼此的根来源。粗略指导下，可以尝试永久代增加浮动垃圾数量的20%大小。堆中每次回收周期结束的浮动垃圾会在下次回收周期被回收。
 ### 暂停
  CMS在一次回收周期中会暂停两次。第一次暂停是为了标记gc roots的直接可达的存活对象（例如，应用线程栈，register，静态对象等引用的对象）和堆中其他地方直接引用（比如年轻代引用）。第一次暂停也被称作initial mark pause（初始标记暂停）。第二次暂停在并发跟踪阶段结束时发生，用于查找在并发跟踪阶段应用线程执行过程中由于对象中引用更新导致的未标记的不可达对象。也被称作remark pause（标记暂停）。

  ### 并发阶段
   并发跟踪可达对象图在初始标记暂停和标记暂停中间发生。该阶段一个或多个gc线程并发执行会使用本属于应用的处理器资源。因此，计算密集型应用在并发跟踪或其他并发阶段可能会相应感受到应用吞吐量降低，虽然此时应用线程并没有被暂停。标记暂停之后，会有一个并发清理阶段回收不可达对象。一旦回收周期结束，cms会等待，等待阶段基本不消耗计算资源，直到下一个major回收周期启动。

 ### 启动并发回收周期
  串行gc的major回收发生在永久代变满时，此时应用线程会停止，直达回收结束。相反，并发回收启动时间必须控制在永久代变满之前，可以完成回收。否则应用会偶遇并发模式失败感受到更长的暂停时间。一下有几个方式启动并发回收。
  * 基于最近历史，cms维护永久代变满前的剩余时间估计值和并发回收周期需要时间。使用这些动态估计值，并发回收周期启动时，尽量保持在永久代枯竭前完成回收。这些估计值为了安全会扩大，因为并发模式失败非常耗时。

  * 并发回收如果永久代占用超过一定初始占用比时也会发生。默认值为大约92%，但该值会随着版本进行调整。该值可以手工调节，使用该配置 **-XX:CMSInitiatingOccupancyFraction=\<N>** 调节。N为0-100的整数，代表永久代大小百分比。

  ### 调度暂停
   年轻代回收的暂停和永久代回收各自独立发生。他们并不重叠，但可能连续发生，导致一个回收暂停后，立即跟着另一个回收的暂停。看起来就像是一个单一的更久的暂停。为了避免这个，CMS会尝试在前一个和下一个年轻代暂停之间，加入一个remark的暂停。这样调度并没提供给初始标记暂停，因为通常这个比标记暂停短。

  ### 增量模式
   注意增量模式在JAVA8中不推荐使用，可能在未来版本移除。
   CMS可以使用在并发阶段增量的模式处理。回想并发阶段gc线程会使用一个或多个处理器。增量模式可以减轻长久的并发阶段影响。这是通过周期性的停止并发阶段，将处理器交回给应用。该模式称作i-cms，将并发的工作分割成多个小块的时间，可以在年轻代回收之间发生。该特性在应用需要较少暂停时间，且机器处理器数量较少（比如1个或2个）时很有用。
   并发回收周期典型包括如下步骤：
   * 停止所有应用线程，标记gc roots的可达对象集合，然后回复所有应用线程
   * 并发跟踪可达对象图，使用一个或多个处理器。辞职应用线程还在执行。
   * 使用一个处理器同时回溯自上一步中以来修改过的对象图的部分。
   * 停止所有应用线程，回溯roots和对象图的上次检查过后部分可能被修改的内容。
   * 使用一个处理器同时清理不可达对象，加入到分配的空闲列表内。
   * 使用一个处理器，调整堆的大小ing准备下一次回收周期的支撑数据结构。
   一般，CMS使用一个或多个处理器在整个并发跟踪阶段，并不会自愿放弃这点。类似的，并发清理阶段只使用一个处理器，也不会放弃这点。在有相应时间约束的应用上，这可能导致太久中断，尤其时在仅有1个或2个处理器的系统上。增量模式通过将并发阶段分解成更多的短暂突发活动，这些活动在minor暂停之间调度。
   i-cms使用任务周期来控制cms在放弃处理器前的回收工作量。任务周期时年轻代回收间允许执行的时间百分比。i-cms可以根据应用程序行为自动计算时间比例（推荐该方式，称作自动调步），或者可以在命令行上设置固定值。
### 命令行选项
 i-cms模式的选项列表如下。后面部分推荐了初始的选项集合：
 > Table 8-1 Command-Line Options for i-cms

 Option|Description|Default Value, Java SE 5 and Earlier|Default Value, Java SE 6 and Later
 ---|---|---|---
 -XX:+CMSIncrementalMode|Enables incremental mode. Note that the CMS collector must also be enabled (with -XX:+UseConcMarkSweepGC) for this option to work.|disabled|disabled
 -XX:+CMSIncrementalPacing|Enables automatic pacing. The incremental mode duty cycle is automatically adjusted based on statistics collected while the JVM is running.|disabled|disabled
 -XX:CMSIncrementalDutyCycle=\<N>|The percentage (0 to 100) of time between minor collections that the CMS collector is allowed to run. If CMSIncrementalPacing is enabled, then this is just the initial value.|50|10
 -XX:CMSIncrementalDutyCycleMin=\<N>|The percentage (0 to 100) that is the lower bound on the duty cycle when CMSIncrementalPacing is enabled.|10|0
 -XX:CMSIncrementalSafetyFactor=\<N>|The percentage (0 to 100) used to add conservatism when computing the duty cycle|10|10
 -XX:CMSIncrementalOffset=\<N>|The percentage (0 to 100) by which the incremental mode duty cycle is shifted to the right within the period between minor collections.|0|0
 -XX:CMSExpAvgFactor=\<N>|The percentage (0 to 100) used to weight the current sample when computing exponential averages for the CMS collection statistics.|25|25

 ### 推荐选项
 java8中使用i-cms，可以用如下选项
 ```
 -XX:+UseConcMarkSweepGC -XX:+CMSIncrementalMode \
 -XX:+PrintGCDetails -XX:+PrintGCTimeStamps
 ```
 java5或更早版本，推荐如下配置作为初始选项：
 ```
 -XX:+UseConcMarkSweepGC -XX:+CMSIncrementalMode \
 -XX:+PrintGCDetails -XX:+PrintGCTimeStamps \
 -XX:+CMSIncrementalPacing -XX:CMSIncrementalDutyCycleMin=0
 -XX:CMSIncrementalDutyCycle=10
 ```

 ### 基本故障定位
  i-cms基于应用过去行为预测未来行为不一定准确。如果发生了太多full gc，尝试下表的选项，一个尝试一个。
 > Table 8-2 Troubleshooting the i-cms Automatic Pacing Feature

Step|options
---|---
1.增加safety factor|-XX:CMSIncrementalSafetyFactor=\<N>
2.增加最小任务周期|-XX:CMSIncrementalDutyCycleMin=\<N>
3.禁用自动调步，使用固定任务周期|-XX:-CMSIncrementalPacing -XX:CMSIncrementalDutyCycle=\<N>

### 测量
 下面样例是cms开启了-verbose:gc和-XX:+PrintGCDetails的输出，删除少量minor细节。注意输出中，CMS回收穿插在minor回收之间。典型的多个minor回收发生在一次并发回收周期内。CMS-initial-mark显示并发回收周期开始，CMS-concurrent-mark显示并发标记阶段结束，CMS-concurrent-sweep标记并发清理阶段结束。CMS-concurrent-preclean显示于清理阶段。Precleaning 前面未提及，代表标记阶段可以并发的准备工作。最后阶段是CMS-concurrent-reset，用于准备下次回收周期。

> Example 8-1 Output from the CMS Collector

```
  [GC [1 CMS-initial-mark: 13991K(20288K)] 14103K(22400K), 0.0023781 secs]
  [GC [DefNew: 2112K->64K(2112K), 0.0837052 secs] 16103K->15476K(22400K), 0.0838519 secs]
  ...
  [GC [DefNew: 2077K->63K(2112K), 0.0126205 secs] 17552K->15855K(22400K), 0.0127482 secs]
  [CMS-concurrent-mark: 0.267/0.374 secs]
  [GC [DefNew: 2111K->64K(2112K), 0.0190851 secs] 17903K->16154K(22400K), 0.0191903 secs]
  [CMS-concurrent-preclean: 0.044/0.064 secs]
  [GC [1 CMS-remark: 16090K(20288K)] 17242K(22400K), 0.0210460 secs]
  [GC [DefNew: 2112K->63K(2112K), 0.0716116 secs] 18177K->17382K(22400K), 0.0718204 secs]
  [GC [DefNew: 2111K->63K(2112K), 0.0830392 secs] 19363K->18757K(22400K), 0.0832943 secs]
  ...
  [GC [DefNew: 2111K->0K(2112K), 0.0035190 secs] 17527K->15479K(22400K), 0.0036052 secs]
  [CMS-concurrent-sweep: 0.291/0.662 secs]
  [GC [DefNew: 2048K->0K(2112K), 0.0013347 secs] 17527K->15479K(27912K), 0.0014231 secs]
  [CMS-concurrent-reset: 0.016/0.016 secs]
  [GC [DefNew: 2048K->1K(2112K), 0.0013936 secs] 17527K->15479K(27912K), 0.0014814 secs
  ]
```
一般初始标记暂停相对minor暂停更短，并发阶段（并发标记，并发预清理，并发清理）会持续比minor暂停更久。然而并发阶段应用是没暂停的。标记阶段通常和minor回收比较。remark暂停会受应用特点（比如对象频繁修改会导致该暂停）影响，并且收上次minor回收的时间影响（比如年轻代对象太多会增加该暂停）。

 ## G1回收器
  G1垃圾回收器是服务端风格的，适用于大量内存的多处理器机器。它会尝试在达成gc暂停时间的目标时，高概率完成高吞吐量。所有堆操作，类似全局标记，都和应用线程并发执行。这可以防止与堆或存活对象大小成比例的中断。


  堆被分区为一组大小相等的堆区域，每个堆区域都是连续的虚拟内存范围。 G1的并发全局标记阶段，查找整个堆中的存活对象。在标记阶段完成之后，G1知道哪些区域几乎是空的。 首先会收集这些空区域，通常会产生大量空闲空间。 这就是为什么这种垃圾收集方法称为Garbage-First。 顾名思义，G1将收集和压缩活动集中在堆的可能充满可回收对象的区域，即垃圾。 G1使用暂停预测模型来满足用户定义的暂停时间目标，基于指定的暂停时间目标来选择对应收集的区域数。

  G1将从堆的一个或多个区域的对象复制到堆上的单个区域，这样压缩并释放内存。 这种清理在多处理器上并行执行，以减少暂停时间并提高吞吐量。 因此，随着每次垃圾收集，G1不断努力减少碎片。 这超出了以前两种方法的能力。 CMS（Concurrent Mark Sweep）垃圾收集不进行压缩。 并行回收器对整堆压缩，这会导致相当长的暂停时间。

  值得注意的是，G1不是实时收集器。 它以高概率但不是绝对确定性满足设定的暂停时间目标。 根据以前收集的数据，G1估计在目标时间内可以收集多少个区域。 因此，收集器具有收集区域的成本的相当准确的模型，并且它使用该模型来确定在停留在暂停时间目标内时要收集哪些区域和多少区域。

  G1的首要目标是给有大堆且需要优先gc时延的应用提供解决方案。具体指有超过6GB的堆大小，且需要稳定可预测的少于0.5秒的暂停时间。

  如果应用有如下一个或多个特征，可以从CMS或并行回收切换到G1：
   * java堆中的存活对象超过50%
   * 对象分配频率或对象晋升的速率差异很大
   * 应用正经历不期望的长时间gc回收或整理暂停（超过0.5秒到1秒）

  G1计划作为Concurrent Mark-Sweep Collector（CMS）的长期替代品。 将G1与CMS进行比较揭示了使G1成为更好解决方案的差异。 一个区别是G1是压缩收集器。 此外，G1提供比CMS收集器更可预测的垃圾收集暂停，并允许用户指定所需的暂停目标。

  > Figure 9-1 Heap Division by G1

  ![Figure 9-1 Heap Division by G1](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/img/jsgct_dt_004_grbg_frst_hp.png)


  G1是逻辑上分代的。一组空区域被指定为逻辑年轻代。在图中，年轻一代是浅蓝色的。对象分配是在逻辑年轻代中完成，当年轻代充满时，这组区域被垃圾收集（一个年轻代回收）。在某些情况下，年轻区域之外的区域（深蓝色的年老代区域）可以同时进行垃圾收集。这被称为混合集合。在该图中，正在收集的区域用红色框标记。该图说明了一个混合回收，因为正在收集年轻区域和年老地区。这个是一个压缩回收，它将活动对象复制到选定的最初为空的区域。基于幸存对象的年龄，可以将对象复制到幸存者区域（标记为“S”）或年老区域（未具体示出）。标有“H”的区域包含超大对象（大小超过区域的一半，所以特殊对待）;请参阅Garbage-First垃圾收集器中的[Humongous Objects和Humongous Allocations部分](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/g1_gc_tuning.html#humongous)。

  ### 分配失败
  和CMS一样，G1的部分回收过程和应用一起运行，这样就有应用分配对象比回收过程更快的风险。可以查看cms的并发模式失败看类似行为。G1中，当G1在从待回收区域复制到其他区域时可能发生类似失败。复制是为了压缩活动对象。如果在这个过程中找不到空闲的区域复制，分配失败就发生了(没有空间可以防止需要复制的对象)，STW的full回收就会发生。

  ### 浮动垃圾
  G1回收过程中可能会有对象变的不可达，因此而未被回收。G1使用类似先快照（SATB:snapshot-at-the-beginning）的技术来保证所有存活对象都能被找到。SATB记录了并发标记开始时的对象都是活动的，这些对象会被认为在回收阶段是活动的。SATB允许浮动垃圾的方式类似于CMS增量更新。

  ### 暂停
  G1会在复制活动对象到新区域时暂停应用。暂停可能是年轻代回收暂停（比如只有年轻区域被收集或者混合收集暂停中包含年轻和年老区域）。和CMS一样，最终有标记或重标记暂停，会暂停应用来完成标记。虽然CMS也有初始标记暂停，但G1会将初始标记工作作为疏散暂停的一部分。 G1在集合的末尾有一个清理阶段，部分是STW，部分是并发的。 清理阶段的STW部分识别空区域并确定作为下一个集合的候选的旧区域。

  ### 卡表和并发阶段
如果垃圾收集器不收集整个堆（增量集合），则垃圾收集器需要知道从堆的未收集部分到正在收集的堆的部分中的指针。 这通常用于分代垃圾收集器，其中堆的未收集部分通常是旧代，并且堆的收集部分是年轻代。 用于保存此信息的数据结构（旧代指针指向年轻代对象）是一个记忆集。 卡表是特定类型的记忆集。 Java HotSpot VM使用字节数组作为卡表。 每个字节称为卡。 卡对应于堆中的一系列地址。 对卡进行Dirtying意味着将字节值更改为脏值; 脏值可能包含从卡的覆盖地址范围内的旧一代到年轻代的新指针。

处理卡意味着查看卡以查看是否存在旧代到年轻代指针并且可能对该信息做某事，例如将其转移到另一数据结构。

G1的并发标记阶段是标记从应用程序中找到的活动对象。并发标记从疏散暂停（此时初始标记工作完成）结束持续到重标记阶段。并发清理阶段将集合清空的区域添加到空闲区域列表中，并清除这些区域的记忆集。此外，根据需要运行并发细化线程来处理已被应用程序写入弄脏的卡表条目，这些条目可能具有跨区域引用。

 ### 启动并发收集周期
 如前所述，一次混合收集中同事收集年轻和年老区域。为了收集年老区域，G1对堆中的活动对象进行了完整标记。这种标记是通过并行标记阶段完成的。当整个Java堆的占用率达到参数InitiatingHeapOccupancyPercent的值时，将启动并发标记阶段。使用命令行选项-XX：InitiatingHeapOccupancyPercent =\<NN>设置此参数的值。 InitiatingHeapOccupancyPercent的默认值为45。

 ### 暂停时间目标
 使用标志MaxGCPauseMillis为G1设置暂停时间目标。 G1使用预测模型来确定在该目标暂停时间内可以完成多少垃圾收集工作。在收集最后，G1选择要在下一次收集中收集的区域。集合将包含年轻区域（其大小的总和决定了逻辑年轻代的大小）。部分是通过选择集合集中的年轻区域的数量，G1控制GC暂停的长度。您可以像在其他垃圾收集器中一样在命令行上指定年轻代的大小，但这样做可能会妨碍G1达到目标暂停时间的能力。除暂停时间目标外，您还可以指定暂停时间段的长度。您可以使用此时间范围（GCPauseIntervalMillis）指定最小变更器使用量以及暂停时间目标。 MaxGCPauseMillis的默认值为200毫秒。 GCPauseIntervalMillis（0）的默认值相当于时间跨度上没有要求。

 ## G1垃圾回收器调优
 如下描述了如何对G1的评估，分析和性能进行调优。
 如垃圾优先垃圾收集器一节所述，G1 GC是区域化和世代垃圾收集器，这意味着Java对象堆（堆）被分成许多大小相等的区域。 启动时，Java虚拟机（JVM）设置区域大小。 区域大小可以从1 MB到32 MB不等，具体取决于堆大小。 目标是不超过2048个区域。 eden，suvivor，年老代是这些地区的逻辑集合，并不是连续的。

 G1 GC具有尝试满足的暂停时间目标（软实时）。在年轻的系列中，G1 GC调整其年轻一代（伊甸园和幸存者大小）以满足软实时目标。有关G1 GC暂停原因以及如何设置暂停时间目标的信息，请参阅垃圾优先垃圾收集器中的暂停和暂停时间目标部分。

在混合集合期间，G1 GC根据混合垃圾收集的目标数量，堆的每个区域中活动对象的百分比以及总体可接受的堆浪费百分比来调整收集的旧区域的数量。

G1 GC通过将活动对象从一组或多组区域（称为集合集（CSet））增量并行复制到一个或多个不同的新区域来减少堆碎片，以实现压缩。目标是尽可能多地回收堆空间，从包含最可回收空间的区域开始，同时尝试不超过暂停时间目标（垃圾优先）。

G1 GC使用独立的记忆集（RSets）来跟踪对区域的引用。独立的RSet支持并行和独立的区域集合，因为只有区域的RSet必须被扫描以进入该区域，而不是整个堆。 G1 GC使用写后屏障来记录对堆的更改并更新RSets。

### GC阶段
除了疏散暂停（参见垃圾 - 垃圾收集器中的分配（疏散）故障）组成世界末日（STW）年轻和混合垃圾收集，G1 GC还具有并行，并发和多相标记周期。 G1 GC使用开始时的快照（SATB）算法，该算法在标记周期开始时逻辑上获取堆中活动对象集的快照。活动对象集还包括自标记周期开始以来分配的对象。 G1 GC标记算法使用预写屏障来记录和标记属于逻辑快照的对象。

### 年轻的垃圾收集
G1 GC满足来自添加到伊甸园区域集的区域的大多数分配请求。在年轻的垃圾收集过程中，G1 GC从之前的垃圾收集中收集伊甸园区域和幸存区域。来自伊甸园和幸存者区域的活体物体被复制或撤离到一组新的区域。特定对象的目标区域取决于对象的年龄;已老化的物体足够疏散到老一代地区（即，它被提升）;否则，该物体撤离到幸存者区域，并将被包括在下一个年轻或混合垃圾收集的CSet中。

### 混合垃圾收集
成功完成并发标记周期后，G1 GC将从执行年轻垃圾收集切换到执行混合垃圾收集。在混合垃圾收集中，G1 GC可选地将一些旧区域添加到将被收集的伊甸园区域和幸存区域中。添加的旧区域的确切数量由许多标志控制（请参阅“建议”部分中的“驯服混合垃圾收集器”）。 G1 GC收集足够数量的旧区域（通过多个混合垃圾收集）后，G1将恢复执行年轻垃圾收集，直到下一个标记周期完成。

### 标记周期阶段
标记周期包括以下阶段：

* 初始标记阶段：G1 GC在此阶段标记根对象。这个阶段捎带在正常（STW）年轻垃圾收集上。

* 根区扫描阶段：G1 GC扫描在初始标记阶段标记的幸存区域，以引用旧代，并标记引用的对象。此阶段与应用程序（而不是STW）同时运行，并且必须在下一个STW年轻垃圾收集开始之前完成。

* 并发标记阶段：G1 GC在整个堆中查找可到达（实时）对象。此阶段与应用程序同时发生，并且可以被STW年轻垃圾收集中断。

* 重标记阶段：此阶段是STW收集，有助于完成标记周期。 G1 GC排出SATB缓冲区，跟踪未访问的活动对象，并执行参考处理。

* 清理阶段：在最后阶段，G1 GC执行会计和RSet清理的STW操作。在会计期间，G1 GC识别完全自由区域和混合垃圾收集候选者。清理阶段在重置并将空区域返回到空闲列表时部分并发。

### 重要的默认值
G1 GC是一个自适应垃圾收集器，默认设置使其无需修改即可高效工作。 表10-1“G1垃圾收集器的重要选项的默认值”Java HotSpot VM，build 24中的重要选项及其默认值列表。您可以通过输入选项来调整和调整G1 GC以满足应用程序性能需求 表10-1“G1垃圾收集器的重要选项的默认值”，其中JVM命令行上的设置已更改。

>Option and Default Value	Option
-XX:G1HeapRegionSize=n
>
>Sets the size of a G1 region. The value will be a power of two and can range from 1 MB to 32 MB. The goal is to have around 2048 regions based on the minimum Java heap size.
>
>-XX:MaxGCPauseMillis=200
>
>Sets a target value for desired maximum pause time. The default value is 200 milliseconds. The specified value does not adapt to your heap size.
>
>-XX:G1NewSizePercent=5
>
>Sets the percentage of the heap to use as the minimum for the young generation size. The default value is 5 percent of your Java heap.Foot1
>
>This is an experimental flag. See How to Unlock Experimental VM Flags for an example. This setting replaces the -XX:DefaultMinNewGenPercent setting.
>
>-XX:G1MaxNewSizePercent=60
>
>Sets the percentage of the heap size to use as the maximum for young generation size. The default value is 60 percent of your Java heap.Footref1
>
>This is an experimental flag. See How to Unlock Experimental VM Flags for an example. This setting replaces the -XX:DefaultMaxNewGenPercent setting.
>
>-XX:ParallelGCThreads=n
>
>Sets the value of the STW worker threads. Sets the value of n to the number of logical processors. The value of n is the same as the number of logical processors up to a value of 8.
>
>If there are more than eight logical processors, sets the value of n to approximately 5/8 of the logical processors. This works in most cases except for larger SPARC systems where the value of n can be approximately 5/16 of the logical processors.
>
>-XX:ConcGCThreads=n
>
>Sets the number of parallel marking threads. Sets n to approximately 1/4 of the number of parallel garbage collection threads (ParallelGCThreads).
>
>-XX:InitiatingHeapOccupancyPercent=45
>
>Sets the Java heap occupancy threshold that triggers a marking cycle. The default occupancy is 45 percent of the entire Java heap.
>
>-XX:G1MixedGCLiveThresholdPercent=85
>
>Sets the occupancy threshold for an old region to be included in a mixed garbage collection cycle. The default occupancy is 85 percent.Footref1
>
>This is an experimental flag. See How to Unlock Experimental VM Flags for an example. This setting replaces the -XX:G1OldCSetRegionLiveThresholdPercent setting.
>
>-XX:G1HeapWastePercent=5
>
>Sets the percentage of heap that you are willing to waste. The Java HotSpot VM does not initiate the mixed garbage collection cycle when the reclaimable percentage is less than the heap waste percentage. The default is 5 percent.Footref1
>
>-XX:G1MixedGCCountTarget=8
>
>Sets the target number of mixed garbage collections after a marking cycle to collect old regions with at most G1MixedGCLIveThresholdPercent live data. The default is 8 mixed garbage collections. The goal for mixed collections is to be within this target number.Footref1
>
>-XX:G1OldCSetRegionThresholdPercent=10
>
>Sets an upper limit on the number of old regions to be collected during a mixed garbage collection cycle. The default is 10 percent of the Java heap.Footref1
>
>-XX:G1ReservePercent=10
>
>Sets the percentage of reserve memory to keep free so as to reduce the risk of to-space overflows. The default is 10 percent. When you increase or decrease the percentage, make sure to adjust the total Java heap by the same amount.Footref1

### 如何解锁实验性VM标志
要更改实验标志的值，您必须先解锁它们。您可以通过在任何实验标志之前在命令行上显式设置-XX：+ UnlockExperimentalVMOptions来完成此操作。例如：
```
java -XX：+ UnlockExperimentalVMOptions -XX：G1NewSizePercent = 10 -XX:G1MaxNewSizePercent = 75 G1test.jar
```


### 建议
在评估和调整G1 GC时，请记住以下建议：

* 年轻代大小：避免使用-Xmn选项或任何或其他相关选项（如-XX：NewRatio）显式设置年轻代大小。修复年轻一代的大小会覆盖目标暂停时间目标。

* 暂停时间目标：评估或调整任何垃圾收集时，始终存在延迟与吞吐量的权衡。 G1 GC是一个增量垃圾收集器，具有统一的暂停，但在应用程序线程上也有更多的开销。 G1 GC的吞吐量目标是90％的应用程序时间和10％的垃圾回收时间。将其与Java HotSpot VM并行收集器进行比较。并行收集器的吞吐量目标是99％的应用程序时间和1％的垃圾收集时间。因此，当您评估G1 GC的吞吐量时，请放宽暂停时间目标。设置过于激进的目标表明您愿意承担垃圾收集开销的增加，这会直接影响吞吐量。当您评估G1 GC的延迟时，您可以设置所需的（软）实时目标，G1 GC将尝试满足它。作为副作用，吞吐量可能受到影响。有关其他信息，请参阅垃圾优先垃圾收集器中的暂停时间目标部分。

* 驯服混合垃圾收集：调整混合垃圾收集时，请尝试以下选项。有关这些选项的信息，请参阅重要默认值部分：

  * -XX：InitiatingHeapOccupancyPercent：用于更改标记阈值。

  * -XX：G1MixedGCLiveThresholdPercent和-XX：G1HeapWastePercent：用于更改混合垃圾收集决策。

  * -XX：G1MixedGCCountTarget和-XX：G1OldCSetRegionThresholdPercent：用于调整旧区域的CSet。

### 溢出和耗尽的日志消息
当您在日志中看到空间溢出或空间耗尽的消息时，G1 GC没有足够的内存用于幸存者或提升的对象，或两者都有。 Java堆不能，因为它已经达到最大值。示例消息：
```
924.897：[GC暂停（G1撤离暂停）（混合）（空间耗尽），0.1957310秒]

924.897：[GC暂停（G1撤离暂停）（混合）（空间溢出），0.1957310秒]
```

要解决此问题，请尝试以下调整：

 * 增加-XX：G1ReservePercent选项的值（以及相应的总堆）以增加“to-space”的保留内存量。

* 通过减少-XX：InitiatingHeapOccupancyPercent的值来提前开始标记周期。

* 增加-XX：ConcGCThreads选项的值以增加并行标记线程的数量。

有关这些选项的说明，请参阅重要默认值部分。

### 超大对象和超大分配
对于G1 GC，任何超过区域大小一半的对象都被视为一个巨大的对象。这样的对象直接在老一代分配到了巨大的区域。这些巨大的地区是一个连续的地区。 StartsHumongous标记连续集的开始，ContinuesHumongous标记集的延续。

在分配任何巨大区域之前，检查标记阈值，如有必要，启动并发周期。

在清理阶段期间以及在完整的垃圾收集循环期间，在标记循环结束时释放死的巨大物体。

为了减少复制开销，任何疏散暂停都不包括大量的物体。完整的垃圾收集周期可以压缩大量的物体。

因为StartsHumongous和ContinuesHumongous区域的每个单独的集合仅包含一个巨大的对象，所以该对象的末端与该对象跨越的最后一个区域的末尾之间的空间未被使用。对于仅略大于堆区域大小的倍数的对象，此未使用的空间可能导致堆碎片化。

如果您看到由于大量分配而启动的背靠背并发周期，并且如果此类分配正在分割您的旧代，则增加-XX：G1HeapRegionSize的值，使之前的大型对象不再是大量的并且将遵循常规分配路径。

## 其他考虑
本节介绍影响垃圾收集的其他情况。

最终确定和弱，软和幻影参考
某些应用程序通过使用finalization和weak，soft或phantom引用与垃圾收集进行交互。这些功能可以在Java编程语言级别创建性能工件。一个例子是依靠finalization来关闭文件描述符，这使得外部资源（描述符）依赖于垃圾收集的快速性。依靠垃圾收集来管理内存以外的资源几乎总是一个坏主意。

前言中的相关文档部分包括一篇文章，深入讨论了最终确定的一些缺陷和避免它们的技术。

显式垃圾收集
应用程序与垃圾收集交互的另一种方法是通过调用System.gc（）显式调用完整垃圾收集。这可以强制在可能没有必要时进行主要收集（例如，当次要收集就足够时），因此通常应该避免。显式垃圾收集的性能影响可以通过使用标志-XX：+ DisableExplicitGC禁用它们来测量，这会导致VM忽略对System.gc（）的调用。

显式垃圾收集最常遇到的用途之一是使用远程方法调用（RMI）的分布式垃圾收集（DGC）。使用RMI的应用程序引用其他虚拟机中的对象。在没有偶尔调用本地堆的垃圾收集的情况下，无法在这些分布式应用程序中收集垃圾，因此RMI会定期强制执行完整收集。可以使用属性控制这些集合的频率，如以下示例所示：

java -Dsun.rmi.dgc.client.gcInterval = 3600000
    -Dsun.rmi.dgc.server.gcInterval = 3600000 ...
此示例每小时指定一次显式垃圾回收，而不是每分钟一次的默认速率。但是，这也可能导致某些物体需要更长的时间才能回收。如果不希望DGC活动的时间性上限，则可以将这些属性设置为与Long.MAX_VALUE一样高，以使显式集合之间的时间实际上无限。

软参考
软引用在服务器虚拟机中比在客户端中保持更长时间。可以使用命令行选项-XX：SoftRefLRUPolicyMSPerMB = <N>来控制清除率，该选项指定每个兆字节的软引用将保持活动状态（一旦不再可以强烈访问）的毫秒数（ms）堆中的可用空间。默认值为每兆字节1000毫秒，这意味着对于堆中的每兆字节可用空间，软引用将在（收集对象的最后一次强引用之后）存活1秒钟。这是一个近似数字，因为软件引用仅在垃圾收集期间被清除，这可能偶尔发生。

类元数据
Java类在Java Hotspot VM中具有内部表示，并称为类元数据。在以前的Java Hotspot VM版本中，类元数据是在所谓的永久生成中分配的。在JDK 8中，永久代被删除，类元数据在本机内存中分配。默认情况下，可用于类元数据的本机内存量是无限制的。使用选项MaxMetaspaceSize为用于类元数据的本机内存量设置上限。

Java Hotspot VM显式管理用于元数据的空间。从操作系统请求空间，然后分成块。类加载器从其块中为元数据分配空间（块被绑定到特定的类加载器）。为类加载器卸载类时，其块将被回收以供重用或返回到操作系统。元数据使用由mmap分配的空间，而不是malloc。

如果启用了UseCompressedOops并且使用了UseCompressedClassesPointers，则将两个逻辑上不同的本机内存区域用于类元数据。 UseCompressedClassPointers使用32位偏移量来表示64位进程中的类指针，对于Java对象引用也使用UseCompressedOops。为这些压缩类指针（32位偏移）分配区域。可以使用CompressedClassSpaceSize设置区域的大小，默认情况下为1千兆字节（GB）。压缩类指针的空间保留为mmap在初始化时分配的空间，并根据需要提交。 MaxMetaspaceSize适用于已提交的压缩类空间和其他类元数据的空间之和。

卸载相应的Java类时，将释放类元数据。由于垃圾收集而卸载Java类，并且可能会导致垃圾收集以卸载类并释放类元数据。当为类元数据提交的空间达到一定水平（高水位线）时，会引发垃圾收集。在垃圾收集之后，可以根据从类元数据中释放的空间量来升高或降低高水位标记。将提高高水位标记，以免过早引起另一次垃圾收集。高水位标记最初设置为命令行选项MetaspaceSize的值。它根据MaxMetaspaceFreeRatio和MinMetaspaceFreeRatio选项升高或降低。如果类元数据的可用空间占类元数据总承诺空间的百分比大于MaxMetaspaceFreeRatio，则高水位标记将降低。如果它小于MinMetaspaceFreeRatio，那么将引发高水位标记。

为MetaspaceSize选项指定更高的值，以避免引发类元数据的早期垃圾收集。为应用程序分配的类元数据量取决于应用程序，并且不存在选择MetaspaceSize的一般准则。 MetaspaceSize的默认大小取决于平台，范围从12 MB到大约20 MB。

有关用于元数据的空间的信息包含在堆的打印输出中。典型输出如例11-1“典型堆打印输出”所示。

Example 11-1 Typical Heap Printout

```
Heap
  PSYoungGen      total 10752K, used 4419K
    [0xffffffff6ac00000, 0xffffffff6b800000, 0xffffffff6b800000)
    eden space 9216K, 47% used
      [0xffffffff6ac00000,0xffffffff6b050d68,0xffffffff6b500000)
    from space 1536K, 0% used
      [0xffffffff6b680000,0xffffffff6b680000,0xffffffff6b800000)
    to   space 1536K, 0% used
      [0xffffffff6b500000,0xffffffff6b500000,0xffffffff6b680000)
  ParOldGen       total 20480K, used 20011K
      [0xffffffff69800000, 0xffffffff6ac00000, 0xffffffff6ac00000)
    object space 20480K, 97% used
      [0xffffffff69800000,0xffffffff6ab8add8,0xffffffff6ac00000)
  Metaspace       used 2425K, capacity 4498K, committed 4864K, reserved 1056768K
    class space   used 262K, capacity 386K, committed 512K, reserved 1048576K

```

在以Metaspace开头的行中，使用的值是用于加载类的空间量。 容量值是当前分配的块中可用于元数据的空间。 提交的值是可用于块的空间量。 保留值是元数据保留（但不一定是已提交）的空间量。 以类空间行开头的行包含压缩类指针的元数据的相应值。

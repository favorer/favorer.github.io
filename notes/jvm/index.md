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


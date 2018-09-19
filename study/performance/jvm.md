### 官方gc调优
* [Tuning Garbage Collection
with the 5.0 Java TM Virtual Machine](http://www.oracle.com/technetwork/java/gc-tuning-5-138395.html#1.1.%20Generations%7Coutline)
* [Java SE Performance at a Glance](http://www.oracle.com/technetwork/java/javase/tech/performance-jsp-141338.html)
* [Java SE 6 HotSpot[tm] Virtual Machine Garbage Collection Tuning](http://www.oracle.com/technetwork/java/javase/gc-tuning-6-140523.html)
* [java8 gc调优](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/toc.html)
* [G1介绍 The Garbage-First Garbage Collector](http://www.oracle.com/technetwork/java/javase/tech/g1-intro-jsp-135488.html)
* [G1调优 Garbage First Garbage Collector Tuning](http://www.oracle.com/technetwork/articles/java/g1gc-1984535.html)
* [jvm xmind](https://www.processon.com/view/589ab3a6e4b0aa37f30069df)
### CollectedHeap
[内存堆管理器GenCollectedHeap的初始化](https://blog.csdn.net/xhh198781/article/details/41173065)
https://blog.csdn.net/xhh198781/article/category/2697643
https://cn.aliyun.com/jiaocheng/306250.html
https://blog.csdn.net/chenbaige/article/details/57115858
https://blog.csdn.net/jcncsdn/article/details/51317302
java8 gc
[java11 zgc](https://zhuanlan.zhihu.com/p/38348775)
https://www.oschina.net/news/97873/jep-333-a-scalable-low-latency-garbage-collector
[java---垃圾回收](http://www.cnblogs.com/w-wfy/p/6415776.html)
[GC root有哪些](https://www.cnblogs.com/w-wfy/p/6415768.html)
[深入理解 Java G1 垃圾收集器](http://blog.jobbole.com/109170/)
[详解CMS垃圾回收机制](http://www.cnblogs.com/littleLord/p/5380624.html)

[TBJMap](https://github.com/alibaba/TBJMap)

[classloader](https://www.cnblogs.com/doit8791/p/5820037.html)
[HotSpot虚拟机对象探秘](http://www.infoq.com/cn/articles/jvm-hotspot)

----------
### gc 结构
safepoint 待学习？
#### gc判断存活
* 引用计数
* 根搜索算法
#### gc基本算法
* 标记-清除Mark-Clean
* 拷贝算法Copy
* 标记-整理算法Mark-Compact
* 分代收集算法
#### gc类型
* 年轻代gc
  * Serial收集器(复制算法)(串行)
  * ParNew收集器(复制算法)（并行）
  * Parallel Scavenge收集器（并行）
* 年老代gc
  * Serial Old收集器（标记-整理算法）(串行)
  * Parallel Old收集器  （使用多线程和“标记－整理”算法）（并行）
  * CMS收集器(标记-清除)(并发)
  * G1收集器(虚拟分代)(标记-整理)

gc实现虽然很多，像 **串行、并行、并发、分代**，但是最基本的算法却只有几种：**引用计数、标记-清除算法、拷贝和整理**，其中拷贝和整理算法还是以标记清除为基础的。

名称|分代|hotspt gc名称|算法类型|并行度|性能目标|常用配套使用|备注|
---|---|---|---|---|---|---|---
Serial|年轻代|Copy|复制算法|串行-单线程||Serial+SerialOld(client默认), Serial+CMS|
ParNew|年轻代|ParNew|复制算法|并行-多线程||ParNew+SerialOld ParNew+CMS|可以和CMS一起用 "ParNew" does the synchronization needed so that it can run during the concurrent phases of CMS.
Parallel Scavenge|年轻代|PS Scavenge|复制算法|并行-多线程|达到可控吞吐量|PS+Serial Old, PS+Parallel Old|PS不能和CMS一起用
||||
Serial Old|年老代|MarkSweepCompact,  "PS MarkSweep(与PS一起用时，叫这个)"|标记-整理算法mark-sweep-compact|串行-单线程||ParNew+Serial Old,  ParNew+Serial Old, Parallel Scavanage+Serial Old cms失败后尝试用Serial Old|
Parallel Old|年老代|PS MarkSweep 这个和Serial Old的一样|标记－整理 |并行|吞吐量|Parallel Scavenge+Parallel Scavenge Old|
CMS|年老代|ConcurrentMarkSweep|标记清除|并行 concurrent mode failure用Serial Old也是串行|低停顿|ParNew+CMS
G1|虚拟分代，年轻年老都有| G1 Young Generation,  G1 Old Generation|标记-整理|并行|低停顿+增量式||标记整理没有内存碎片问题，精确控制停顿时间

* [JVM学习笔记九 之 GC（对象的生命周期系列）](http://yueyemaitian.iteye.com/blog/1185301)
* [第3章　垃圾收集器与内存分配策略](https://blog.csdn.net/wisgood/article/details/16368551)
* [关于GC参数的问题，介绍了gc名称](http://hllvm.group.iteye.com/group/topic/27629)
* [ParNew 和 PSYoungGen 和 DefNew 是一个东西么？](http://hllvm.group.iteye.com/group/topic/37095)
* [JVM调优——之CMS GC日志分析](https://www.cnblogs.com/onmyway20xx/p/6590603.html)
* [JVM中的STW和CMS](http://www.cnblogs.com/williamjie/p/9222839.html)

 实战
* [gc监控不错：一次JVM报警 Cocurrence Mark Sweep 报警](https://blog.csdn.net/vbaspdelphi/article/details/52859882)
* [一次线上JVM调优实践，FullGC40次/天到10天一次的优化过程](https://blog.csdn.net/cml_blog/article/details/81057966)
* [jvm metaspace导致FGC](https://blog.csdn.net/zjwstz/article/details/77478054)
* [如何避免后台IO高负载造成的长时间JVM GC停顿(转)](https://www.cnblogs.com/williamjie/p/9222836.html)
* baidu 搜索PSScavenge PSMarkSweep可以看到很多内容

#### Parallel Scavenge收集器
由于与吞吐量关系密切，Parallel Scavenge收集器也经常被称为“吞吐量优先”收集器。除上述两个参数之外，Parallel Scavenge收集器还有一个参数-XX:+UseAdaptiveSizePolicy值得关注。这是一个开关参数，当这个参数打开之后，就不需要手工指定新生代的大小（-Xmn）、Eden与Survivor区的比例（-XX:SurvivorRatio）、晋升老年代对象年龄（-XX:PretenureSizeThreshold）等细节参数了，虚拟机会根据当前系统的运行情况收集性能监控信息，动态调整这些参数以提供最合适的停顿时间或最大的吞吐量，这种调节方式称为GC自适应的调节策略（GC Ergonomics）。如果读者对于收集器运作原理不太了解，手工优化存在困难的时候，使用Parallel Scavenge收集器配合自适应调节策略，把内存管理的调优任务交给虚拟机去完成将是一个很不错的选择。只需要把基本的内存数据设置好（如-Xmx设置最大堆），然后使用MaxGCPauseMillis参数（更关注最大停顿时间）或GCTimeRatio参数（更关注吞吐量）给虚拟机设立一个优化目标，那具体细节参数的调节工作就由虚拟机完成了。自适应调节策略也是Parallel Scavenge收集器与ParNew收集器的一个重要区别。

#### CMS收集器
阶段|并行度|是否STW|功能
---|---|---|---
初始标记（CMS initial mark）|单线程串行|STW|标记GC Roots能直接关联到的对象
并发标记（CMS concurrent mark）|多线程并发|和用户线程一起执行|并发搜索，查找存活对象
重新标记（CMS remark）|多线程并发|STW|修正并发标记时间，存活对象变化的标记
并发清除（CMS concurrent sweep）|多线程并发|和用户线程一起执行|并发清理不可达对象

* [关于CMS、G1垃圾回收器的重新标记、最终标记疑惑?](https://www.zhihu.com/question/37028283)

缺点：
* CMS收集器对CPU资源非常敏感。
* CMS收集器无法处理浮动垃圾（Floating Garbage）：并发清理阶段继续产生浮动垃圾,CMS无法在本次收集中处理掉它们，只好留待下一次GC时再将其清理掉。
  * 在默认设置下，CMS收集器在老年代使用了68%的空间后就会被激活，这是一个偏保守的设置，如果在应用中老年代增长不是太快，可以适当调高参数-XX:CMSInitiatingOccupancyFraction的值来提高触发百分比，以便降低内存回收次数以获取更好的性能。要是CMS运行期间预留的内存无法满足程序需要，就会出现一次“Concurrent Mode Failure”失败，这时候虚拟机将启动后备预案：临时启用Serial Old收集器来重新进行老年代的垃圾收集，这样停顿时间就很长了。所以说参数-XX:CMSInitiatingOccupancyFraction设置得太高将会很容易导致大量“Concurrent Mode Failure”失败，性能反而降低。
* 空间碎片问题.
  解决方式
  * -XX:+UseCMSCompactAtFullCollectionFull。GC服务之后额外免费附送一个碎片整理过程。空间碎片问题没有了，但停顿时间不得不变长了
  * -XX: CMSFullGCsBeforeCompaction。执行多少次不压缩的Full GC后，跟着来一次带压缩的。

----
### 问题排查工具
# 1.问题排查

## BTrace 系

- [btrace](https://github.com/btraceio/btrace)
- [greys](https://github.com/oldmanpushcart/greys-anatomy) 交互式免脚本，比btrace更易用
- [arthas](https://github.com/alibaba/arthas) Alibaba Java Diagnostic Tool Arthas/Alibaba Java诊断利器Arthas


[Java神器Btrace，从入门到熟练小工手册](https://mp.weixin.qq.com/s/4bZ6iSvpqPsjdvkSoFVhrg)

## 在线日志分析

- [easygc.io](http://www.gceasy.io/) gc日志分析
- [fastthread.io](http://fastthread.io/) thread dump分析

## HeapDump分析

- [Eclipse MAT](https://www.eclipse.org/mat/)

# 2.性能Profile

- [Java Mission Control](http://www.oracle.com/technetwork/java/javaseproducts/mission-control/index.html) JDK自带Profiler
- [async-profiler](https://github.com/jvm-profiling-tools/async-profiler) 火焰图生成工具

### 性能工具分析
* [PerfData](https://www.jianshu.com/p/c784e2535855) jvm统计信息会有一块内存放置，默认mmap映射文件到内存中。jstat等通过DirectByteBuffer直接访问。
  * [direct 和mappedbuffer pool的区别](https://stackoverflow.com/questions/15657837/what-is-mapped-buffer-pool-direct-buffer-pool-and-how-to-increase-their-size)
  * [jstat -snap \<pid>](http://rednaxelafx.iteye.com/blog/796343) 展示进程的perf信息
-----
### jvm源码结构

[代码结构](https://blog.csdn.net/l_215851356/article/details/78624933)
[源码分析](https://github.com/codefollower/OpenJDK-Research/tree/master/hotspot/my-docs)


### jvm源码编译

* [参考过程1](http://www.cnblogs.com/lighten/p/5906359.html)
* [参考过程2](https://www.jianshu.com/p/e85f93cc74cb)
* [参考过程3](http://www.it610.com/article/3532039.htm)
* [参考过程4](https://blog.csdn.net/lpwstr/article/details/78840188)
-----
  过程中主要遇到了几个问题
* freetype安装，需要从github上下载windows的包。配置加上参数--with-freetype="/cygdrive/F\tools\freetype-windows-2.9.1"。
* boot jdk需要使用java7。配置加上--with-boot-jdk="/cygdrive/D\Program Files\Java\jdk1.7.0_15"
* cvtres.exe多个版本重复，导致link时报corrupt。具体见 [error LNK112:fatal error LNK1123: failure during conversion to COFF: file invalid or corrupt](https://www.cnblogs.com/poissonnotes/p/4424296.html)。
  解决方法，将Microsoft Visual Studio 10.0里的2个这个exe改名成-old，从而使用framework里的。
* 工程生成时，create后面跟着的路径是最后镜像输出的路径，可以放到根目录的build里，而不是工程文件的目录。vs工程路径会放在hotspot的build里。
* grep版本由于不匹配遇到的bug。grep编1.8时，需要是2.27-2，我的是3.02。参见 [JDK-8179675](https://bugs.openjdk.java.net/browse/JDK-8179675)
  * cygwin需要安装老版本软件时，点击版本号或版本号旁边，可以选择版本号。
* 模块编译失败后，可以make clean-<module>，再继续make all。
* 如果遇到下面错误可以加上--disable-ccache试下。还需要make clean-<module>，然后继续make all。
```
## Starting jdk
”，由“/cygdrive/f/study/jdk8/build/windows-x86_64-normal-server-release/jdk/objs/libfdlibm/e_acos.obj” 需求。 停止。
```
```
  bash ./configure --with-freetype="/cygdrive/F\tools\freetype-windows-2.9.1" --with-boot-jdk="/cygdrive/D\Program Files\Java\jdk1.7.0_15" --disable-ccache
```

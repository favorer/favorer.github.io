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

名称|分代|算法类型|并行度|性能目标|常用配套使用|
---|---|---|---|---|---
Serial|年轻代|复制算法|串行||Serial+SerialOld(client默认), Serial+CMS|
ParNew|年轻代|复制算法|并行||ParNew+SerialOld ParNew+CMS|
Parallel Scavenge|年轻代||并行|达到可控吞吐量|PS+Serial Old, PS+Parallel Old|
||||
Serial Old|年老代|标记-整理算法|串行||ParNew+Serial Old,  ParNew+Serial Old, Parallel Scavanage+Serial Old cms失败后尝试用Serial Old|
Parallel Old|年老代||并行|吞吐量|Parallel Scavenge+Parallel Scavenge Old|
CMS|年老代|标记清除|并行|低停顿|ParNew+CMS
G1|虚拟分代，年轻年老都有|标记-整理|并行|增量式?|

[JVM学习笔记九 之 GC（对象的生命周期系列）](http://yueyemaitian.iteye.com/blog/1185301)
[第3章　垃圾收集器与内存分配策略](https://blog.csdn.net/wisgood/article/details/16368551)
[关于GC参数的问题](http://hllvm.group.iteye.com/group/topic/27629)


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

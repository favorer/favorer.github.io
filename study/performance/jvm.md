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
### 问题排查工具
# 1.问题排查

## BTrace 系

- [btrace](https://github.com/btraceio/btrace)
- [greys](https://github.com/oldmanpushcart/greys-anatomy) 交互式免脚本，比btrace更易用

[Java神器Btrace，从入门到熟练小工手册](https://mp.weixin.qq.com/s/4bZ6iSvpqPsjdvkSoFVhrg)

## 在线日志分析

- [easygc.io](http://www.gceasy.io/) gc日志分析
- [fastthread.io](http://fastthread.io/) thread dump分析

## HeapDump分析

- [Eclipse MAT](https://www.eclipse.org/mat/)

# 2.性能Profile

- [Java Mission Control](http://www.oracle.com/technetwork/java/javaseproducts/mission-control/index.html) JDK自带Profiler
- [async-profiler](https://github.com/jvm-profiling-tools/async-profiler) 火焰图生成工具
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

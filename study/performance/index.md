* [零拷贝：dma,rdma](https://www.ibm.com/developerworks/library/j-zerocopy/)

* 异步
    * java异步特性：CompletableFuture
    * reactivex:rxjava,rxnetty
    * java8 语法 函数等简写
* 高并发包
  * [LongAddr](https://blog.csdn.net/zqz_zqz/article/details/70665941), DoubleAddr解决[伪共享(False Sharing)](http://ifeve.com/falsesharing/)
  * [arraydeque](https://www.cnblogs.com/wxd0108/p/7366234.html)
* [Collection.toArray(new T[0])用0数组更快](https://shipilev.net/blog/2016/arrays-wisdom-ancients/)
* [基准测试jmh](http://openjdk.java.net/projects/code-tools/jmh/)
* [谈谈ConcurrentHashMap1.7和1.8的不同实现](https://www.jianshu.com/p/e694f1e868ec)
* [彻底理解synchronized](https://juejin.im/post/5ae6dc04f265da0ba351d3ff)
* [红黑树](http://www.cnblogs.com/skywang12345/p/3245399.html)
---------------
指标：
[the four golden signals](https://blog.netsil.com/the-4-golden-signals-of-api-health-and-performance-in-cloud-native-applications-a6e87526e74)

框架性能优势
* 连接池
[HikariCP](https://github.com/brettwooldridge/HikariCP/blob/dev/documents/Welcome-To-The-Jungle.md)
[Hikari连接池大小配置](https://juejin.im/post/5ad44c9b6fb9a028d70112e0)
[flexy-pool](https://github.com/vladmihalcea/flexy-pool)
[Flexy Pool, reactive connection pooling](https://www.javacodegeeks.com/2014/04/flexy-pool-reactive-connection-pooling.html)
[连接池选择](https://techblog.topdesk.com/coding/choosing-a-database-connection-pool/)
[连接池定位](http://3ms.huawei.com/km/blogs/details/5465199)

* 无锁算法
[scalable-stack](https://www.cs.bgu.ac.il/~hendlerd/papers/scalable-stack.pdf)
* 性能比较
[zuul vs nginx](https://instea.sk/2015/04/netflix-zuul-vs-nginx-performance/)

* [tomcat调优](http://www.360doc.com/content/16/0413/15/16915_550287724.shtml)
[tomcat调优](https://www.cnblogs.com/lirenzuo/p/7580104.html)
https://www.cnblogs.com/sunfenqing/p/7339058.html

* [spring boot性能优化](https://blog.csdn.net/github_32521685/article/details/50463895)

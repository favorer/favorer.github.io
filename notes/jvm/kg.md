知识整理：
云原生：
	helm： https://helm.sh/zh/docs/chart_template_guide/control_structures/ https://helm.sh/zh/docs/chart_template_guide/getting_started/
			https://blog.csdn.net/m0_48267651/article/details/106452153
			https://blog.csdn.net/u011501223/article/details/107561559
			https://www.jianshu.com/p/d0b0dcad531e
			
	netflix 流量图
	vpc： https://support.huaweicloud.com/productdesc-vpcep/vpcep_01_0001.html
	Init Container(初始化容器) https://blog.csdn.net/zhangshaohuas/article/details/107435394

分布式框架，算法：
			
分布式限流：分布式滑动窗口 分布式限流Sentinel
	https://baijiahao.baidu.com/s?id=1667474456667206690&wfr=spider&for=pc
	亿级流量架构怎么做资源隔离？写得太好了！  https://www.cnblogs.com/javastack/p/15251505.html
cap理论和base理论：https://baike.baidu.com/item/CAP%E7%90%86%E8%AE%BA/5305578?fr=aladdin
分布式锁，分布式协调：
	ShedLock：https://github.com/lukas-krecan/ShedLock
			  https://xie.infoq.cn/article/ca0dd0b69f202a17891e1b369
	
分布式事务：https://www.cnblogs.com/crazymakercircle/p/14375424.html
		https://www.infoq.cn/article/THMgMFVQvpWp9yrrxpMW
		https://www.cnblogs.com/mayundalao/p/11798502.html
	seata： https://seata.io/zh-cn/docs/dev/seata-mertics.html
			https://github.com/seata/seata-samples
			
	
分库分表：sharding-jdbc（当当）TSharding（蘑菇街）Atlas（奇虎360）Cobar（阿里巴巴）MyCAT（基于Cobar）Oceanus（58同城）Vitess（谷歌）
	shardingsphere： https://baijiahao.baidu.com/s?id=1678946491374078281&wfr=spider&for=pc
			https://www.jianshu.com/p/9a346a7bc20d
			https://blog.csdn.net/lsqingfeng/article/details/110423931
			https://baijiahao.baidu.com/s?id=1708720840919615305&wfr=spider&for=pc
			https://blog.csdn.net/crazymakercircle/article/details/123420859
分布式案例：
	分布式架构在云南红塔银行核心系统的实践；https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_9919844620823300757%22%7D&n_type=1&p_from=4
	曹操专车中台 http://lihuia.com/%e6%9b%b9%e6%93%8d%e4%b8%93%e8%bd%a6%e4%b8%ad%e5%8f%b0%e6%9c%8d%e5%8a%a1%e6%b5%8b%e8%af%95%e5%b7%a5%e4%bd%9c%e6%80%bb%e7%bb%93/
	
	
分布式中间件：
	redis：https://www.cnblogs.com/qqflying/p/9192331.html
			redis windows：https://github.com/tporadowski/redis/releases
			https://www.freesion.com/article/282841557/
			https://github.com/redisson/redisson/wiki/%E7%9B%AE%E5%BD%95
			https://www.cnblogs.com/better-farther-world2099/articles/15216447.html
			https://juejin.cn/post/7054485842735136798
			https://blog.csdn.net/Angry_Mills/article/details/119572202
			redisson，lettuce https://github.com/lettuce-io/lettuce-core/issues/1254
			https://github.com/Shawyeok  https://blog.csdn.net/zzhongcy/article/details/102584028
	redisson延迟队列原理：		https://zhuanlan.zhihu.com/p/343811173
			https://mp.weixin.qq.com/s/WcEAmUjtzOLRfGTeKPvrvg
			
	主从选举：https://www.jianshu.com/p/786dcb08e2a0
	redis hash 过大：
	redis源码分析：https://blog.csdn.net/weixin_39932692/article/details/111296115
分布式存储：
	Dynamo，Dynomite，dynoClient： https://www.infoq.cn/article/2014/11/dynomite-netflix-dynamo
				https://www.icode9.com/content-1-944812.html
				https://github.com/Netflix/dyno
				http://www.xjishu.com/zhuanli/62/201610872590.html
分布式消息：
	kafka： https://www.jianshu.com/c/0c9d83802b0c https://docs.spring.io/spring-kafka/docs/current/reference/html/#preface
	kafka监控工具：https://mp.weixin.qq.com/s/6MR7iSDyrtA_KFyhOeE6lA
	kafka stream：https://www.cnblogs.com/tree1123/p/11457851.html
	kafka dlt： https://www.jianshu.com/p/8498ee18d993
		https://www.cnblogs.com/songanwei/p/9202803.html
	
任务编排：
	netflix conductor： https://www.jianshu.com/p/4eae1af8afa8 https://github.com/Netflix/conductor
		https://cloud.tencent.com/developer/article/1367734
	Terraform：https://help.aliyun.com/document_detail/91285.html
	azkaban：https://azkaban.readthedocs.io/en/latest/
	dolphinscheduler：https://github.com/apache/dolphinscheduler/
		
JVM-锁，并发，线程池：
	https://zhuanlan.zhihu.com/p/371008871
	https://blog.csdn.net/yanyan19880509/article/details/52435135
	https://blog.csdn.net/weixin_36904568/article/details/90743847
	https://blog.csdn.net/cmj962464/article/details/94736232
	https://www.jianshu.com/p/2b7d853322bb
	https://blog.csdn.net/weixin_43888181/article/details/116546374
	
JVM-调参:
	https://shimo.im/docs/t8kXpPggYk3KRp9T/read
	https://shimo.im/docs/e1Az4KGg9gs7ZGqW/read
	java gc 暂停时间
	https://juejin.cn/post/6844903808225509390
	gc日志：https://www.cnblogs.com/zhjh256/p/12077117.html
	https://blog.csdn.net/diaokai2608/article/details/101488462
	https://blog.csdn.net/z390174504/article/details/103926003
JVM-编码：
	https://blog.csdn.net/x_iya/article/details/77482046
内存：	https://pdai.tech/md/java/jvm/java-jvm-x-introduce.html
	https://cloud.tencent.com/developer/article/1439628
nio bytebuffer： https://www.zhihu.com/question/60892134
				https://blog.csdn.net/u013096088/article/details/78774627
				

大数据、nosql：
es：https://wenku.baidu.com/view/7d727bf03286bceb19e8b8f67c1cfad6195fe98f.html
	https://wenku.baidu.com/view/12871ef483eb6294dd88d0d233d4b14e85243eed.html
	https://blog.csdn.net/ZYC88888/article/details/83016513
	https://www.pdai.tech/md/db/nosql-es/elasticsearch-x-index-template.html
	https://www.bilibili.com/read/cv11537651
	https://www.cnblogs.com/lyy-blog/p/10195251.html
	https://hackolade.com/help/Elasticsearch.html
	
	elk，efk https://wsgzao.github.io/post/efk/ https://blog.csdn.net/yunwei_gtq/article/details/119323974
			https://elkguide.elasticsearch.cn/ https://elasticsearch.bookhub.zone/#/index_modules/index_modules
			
	
opendistro：	https://opendistro.github.io/for-elasticsearch-docs/docs/im/ism/policies/

hbase：https://hbase.apache.org/2.2/book.html
		https://blog.csdn.net/kangkangwanwan/article/details/89332536

分布式时序数据库：
	DolphinDB https://baike.baidu.com/item/DolphinDB/56163819
列式数据库;
	clickhouse:
Presto： https://tech.meituan.com/2014/06/16/presto.html https://blog.csdn.net/u011596455/article/details/86558218
Kylin：https://zhuanlan.zhihu.com/p/104466978
flink： flink 调试 dump	
	一个flink作业的调优 https://www.cnblogs.com/029zz010buct/p/9774295.html
	一次线上Flink 背压情况分析之重新认识java dump 文件：https://blog.csdn.net/weixin_43291055/article/details/121558923
	flink-dump-fullgc log打印分析 https://blog.csdn.net/u013086392/article/details/105834496
	Flink-复杂事件（CEP） https://zhuanlan.zhihu.com/p/43448829
	

微服务框架：
	cse： https://bbs.huaweicloud.com/blogs/115961
		  http://3ms.huawei.com/km/blogs/details/8606663
		  https://servicecomb.apache.org/cn/docs/users/service-configurations/#%E9%99%90%E6%B5%81%E7%AD%96%E7%95%A5
		  https://my.oschina.net/u/4146444/blog/3060239
	dsf:  http://3ms.huawei.com/hi/group/3216191/wiki_4949441.html
微服务样例开源框架：
	SpringBlade： https://gitee.com/smallc/SpringBlade
	
linux/docker： 
	VIRT 内存 https://blog.csdn.net/qq_33864656/article/details/78286855
	https://blog.csdn.net/u012271526/article/details/118656996
	kmem，anon-rss，total-vm：https://blog.csdn.net/shuihupo/article/details/80905641 https://www.codenong.com/18845857/ https://www.cnblogs.com/fourstupidguns/p/11073616.html
	oom killer： https://www.cnblogs.com/weifeng1463/p/10174432.html
	ps 使用：https://www.cnblogs.com/hml-blog-com/p/11558369.html
	umask：https://blog.csdn.net/yangzhengquan19/article/details/83055686 https://www.runoob.com/linux/linux-comm-umask.html
	进入容器： https://www.cnblogs.com/xhyan/p/6593075.html
	overlay： docker overlay var/lib/docker/overlay2 https://blog.csdn.net/zhonglinzhang/article/details/80970411
			https://blog.csdn.net/haohaoxuexiyai/article/details/111244328
			https://blog.csdn.net/jabony/article/details/104667678
			https://blog.csdn.net/xu710263124/article/details/115553657
	/etc/ld.so.conf.d/目录下文件的作用 https://www.jianshu.com/p/569b8cab106e		
	
linux/网络：
	nat：https://www.zhihu.com/question/40738584?sort=created
		 https://blog.csdn.net/chenglanche9990/article/details/100987459
		 https://www.cnblogs.com/yekainew/archive/2012/12/29/2839431.html?utm_source=tuicool&utm_medium=referral
		 https://www.cnblogs.com/kegeyang/archive/2013/01/27/2879043.html
	net-tools和iproute2
	无类型域间选路(CIDR)
	关于 /dev/(tcp|udp)/${HOST}/${PORT} echo /dev/tcp/ip/port   https://blog.csdn.net/michaelwoshi/article/details/101107042
	
linux：时间 https://tieba.baidu.com/p/2378109648
			https://1335402049.github.io/2021/04/13/%E8%A7%A3%E5%86%B3Kubernetes%E7%9A%84Pod%E4%B8%8E%E5%AE%BF%E4%B8%BB%E6%9C%BA%E6%97%B6%E5%8C%BA%E4%B8%8D%E5%90%8C%E6%AD%A5%E9%97%AE%E9%A2%98/
			https://www.cnblogs.com/leozhanggg/p/14200195.html
tls协议：https://blog.csdn.net/mrpre/article/details/77867439
		 https://www.cnblogs.com/xdyixia/p/11610102.html
		 
		
设计模式，架构模式：
架构师之路 — 软件架构 — 应用架构设计模式 https://is-cloud.blog.csdn.net/article/details/122783342
	https://chowdera.com/2022/02/202202060234549388.html https://bbs.huaweicloud.com/blogs/330329
		 
mysql、mariadb内存分析优化
mariadb 内存占用优化 https://juejin.cn/post/6844903766039216142
	explain：https://zhuanlan.zhihu.com/p/149807046
			 https://blog.csdn.net/yhl_jxy/article/details/88570154
			 https://www.jianshu.com/p/8fab76bbf448

java性能优化;
arthas: 一图xmind https://github.com/alibaba/arthas/issues/1003 
	https://www.yuque.com/arthas-idea-plugin/help/iisg20
	https://arthas.aliyun.com/doc/quick-start.html
	https://www.jianshu.com/p/0ed3c885eb7e
	https://blog.csdn.net/wlchina123/article/details/108679806
	
skywalking: https://skywalking.apache.org/docs/#SkyWalking	
        agent对接：https://github.com/apache/skywalking-java/blob/main/docs/en/setup/service-agent/java-agent/Application-toolkit-micrometer.md
		http://learn.lianglianglee.com/%E4%B8%93%E6%A0%8F/%E5%88%86%E5%B8%83%E5%BC%8F%E9%93%BE%E8%B7%AF%E8%BF%BD%E8%B8%AA%E5%AE%9E%E6%88%98-%E5%AE%8C/18%20%20%E8%A7%82%E6%B5%8B%E5%88%86%E6%9E%90%EF%BC%9ASkyWalking%20%E5%A6%82%E4%BD%95%E6%8A%8A%E8%A7%82%E6%B5%8B%E5%92%8C%E5%88%86%E6%9E%90%E7%BB%93%E5%90%88%E8%B5%B7%E6%9D%A5%EF%BC%9F.md
jvm：	https://www.cnblogs.com/lemon-flm/p/11775893.html
		https://blog.csdn.net/weixin_31207903/article/details/112897140
		https://blog.csdn.net/warrah/article/details/120999863
		
jdb：	https://www.cnblogs.com/qq931399960/p/11316684.html

开源组件：
easyexcel：https://segmentfault.com/a/1190000020791982
log4j： https://www.cnblogs.com/yeyang/p/7944899.html
		https://docs.sentry.io/platforms/java/guides/log4j2/configuration/sampling/
spring:
		Lifecycle https://www.cnblogs.com/deityjian/p/11296846.html
		hbase: https://blog.csdn.net/chinabestchina/article/details/109146489
		加载顺序：https://blog.csdn.net/swq463/article/details/86705925
drools： https://openrules.com/sandbox.htm
icu4j：https://www.jianshu.com/p/3110c69389e7
		
		
python：
metaclass： http://c.biancheng.net/view/2293.html
	https://www.liaoxuefeng.com/wiki/1016959663602400/1017592449371072
	https://www.jianshu.com/p/51b728e0bca9
	https://www.cnblogs.com/huchong/p/8260151.html
	https://python3-cookbook.readthedocs.io/zh_CN/latest/c09/p13_using_mataclass_to_control_instance_creation.html
	https://segmentfault.com/q/1010000007818814
	https://www.cnblogs.com/chengd/articles/7287528.html
	
it新闻网站：
	美团技术：https://tech.meituan.com/
				https://awps-assets.meituan.net/mit-x/2018-ebook-bundle1/2018-ebook-backend.pdf
			  https://tool.lu/article?sort=latest&page=2
			  
	RednaxelaFX ：https://www.zhihu.com/people/rednaxelafx
	https://my.oschina.net/u/945874/blog/371562
	https://www.runoob.com/w3cnote/github-tools.html
	图解芯片技术： https://cread.jd.com/read/startRead.action?bookId=30609289&readType=1
	
物联网：
thingsboard：https://github.com/thingsboard/thingsboard
	http://www.ithingsboard.com/docs/user-guide/rule-engine-2-0/architecture/#actor%E6%A8%A1%E5%9E%8B
	https://max.book118.com/html/2021/1021/6005032111004031.shtm
	
类图设计：https://plantuml.com/zh/class-diagram
springboot 获取git版本	

算法：	
	日志异常检测：https://my.oschina.net/yunzhihui/blog/5595234
	全文索引原理  https://blog.csdn.net/shb_derek1/article/details/77478691
学习：
    java: https://blog.csdn.net/longzhutengyue/article/details/95534447
		  https://baijiahao.baidu.com/s?id=1701167160625876534&wfr=spider&for=pc
		  https://mdnice.com/writing/8c0c3090373641e286c2ec72396c2c17
		  
	mock: https://www.jianshu.com/p/7d602a9f85e3
can分析：
		https://www.zhihu.com/question/48529790
		Vehicle Spy：https://zhuanlan.zhihu.com/p/126660806
		https://github.com/tomyqg/CANoe-asc2csv
		https://github.com/search?p=5&q=can+asc&ref=opensearch&type=Repositories
		https://www.cnblogs.com/zhyongquan/category/1209067.html
		
	UDS: https://blog.whatsroot.xyz/2019/03/02/UDS-DTC-introduction/
	
		
	
	
	

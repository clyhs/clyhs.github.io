---
title: jvm垃圾回收与调优
date: 2019-05-30 15:58:13
category: java
tags: java
---

### jvm的模型

JVM堆内存被分为两部分**年轻代**（Young Generation）、**老年代**（Old Generation）和**永久保存区**（Perm Generation）。

![img](https://clyhs.github.io/images/java/jvm.png)

#### 年轻代

年轻代是所有新对象产生的地方。当年轻代内存空间被用完时，就会触发垃圾回收。这个垃圾回收叫做**Minor GC**。年轻代被分为3个部分**Enden区**和两个**Survivor区**。

年轻代空间的要点：

- 大多数新建的对象都位于Eden区。
- 当Eden区被对象填满时，就会执行Minor GC。并把所有存活下来的对象转移到其中一个survivor区。
- Minor GC同样会检查存活下来的对象，并把它们转移到另一个survivor区。这样在一段时间内，总会有一个空的survivor区。
- 经过多次GC周期后，仍然存活下来的对象会被转移到年老代内存空间。通常这是在年轻代有资格提升到年老代前通过设定年龄阈值来完成的。

#### 年老代

年老代内存里包含了长期存活的对象和经过多次**Minor GC**后依然存活下来的对象。通常会在老年代内存被占满时进行垃圾回收。老年代的垃圾收集叫做**Major GC**。Major GC会花费更多的时间。

#### 永久区

用于存放“永久”对象。这些对象管理着运行于JVM中的类和方法。



### jvm的垃圾回收

* **Serial GC**

-XX:+UseSerialGC：Serial GC使用简单的**标记、清除、压缩**方法对年轻代和年老代进行垃圾回收，即Minor GC和Major GC。Serial GC在client模式（客户端模式）很有用，比如在简单的独立应用和CPU配置较低的机器

* **Parallel GC**

-XX:+UseParallelGC：除了会产生N个线程来进行年轻代的垃圾收集外，Parallel GC和Serial GC几乎一样。这里的N是系统CPU的核数。我们可以使用 -XX:ParallelGCThreads=n 这个JVM选项来控制线程数量

* **CMS GC** 

-XX:+UseConcMarkSweepGC：CMS收集器也被称为短暂停顿并发收集器。它是对年老代进行垃圾收集的。CMS收集器通过多线程并发进行垃圾回收，尽量减少垃圾收集造成的停顿。CMS收集器对年轻代进行垃圾回收使用的算法和Parallel收集器一样。这个垃圾收集器适用于不能忍受长时间停顿要求快速响应的应用。可使用 -XX:ParallelCMSThreads=n JVM选项来限制CMS收集器的线程数量。

* **G1 GC**

-XX:+UseG1GC) G1（Garbage First）：垃圾收集器是在Java 7后才可以使用的特性，它的长远目标时代替CMS收集器。G1收集器是一个并行的、并发的和增量式压缩短暂停顿的垃圾收集器。G1收集器和其他的收集器运行方式不一样，不区分年轻代和年老代空间。它把堆空间划分为多个大小相等的区域

*一般用得最多是cms gc 和G1 gc*

### GC的参数解释



**-XX:GCTimeRatio** 垃圾收集时间占总时间的比率，如果把此参数设置为19，那允许的最大GC时间就占总时间的5%（即1/（1+19）），默认值为99，就是允许最大1%（即1/（1+99））的垃圾收集时间。

**-XX:MaxTenuringThreshold** 晋升老年代的最大年龄。默认为15，比如设为10，则对象在10次普通GC后将会被放入年老代

**-XX:+UseConcMarkSweepGC** 开启CMS垃圾回收器

**-XX:CMSInitiatingOccupancyFraction** 触发CMS收集器的内存比例。比如70%的意思就是说，当内存达到70%，就会开始进行CMS并发收集 

**-XX:CMSFullGCsBeforeCompaction** 设置在几次CMS垃圾收集后，触发一次内存整理

**-XX:+ExplicitGCInvokesConcurrent** System.gc()可以与应用程序并发执行。

**-XX:+UseCMSInitiatingOccupancyOnly** 标志来命令JVM不基于运行时收集的数据来启动CMS垃圾收集周期。而是，当该标志被开启时，JVM通过CMSInitiatingOccupancyFraction的值进行每一次CMS收集，而不仅仅是第一次

**-XX:+CMSClassUnloadingEnabled** 相对于并行收集器，CMS收集器默认不会对永久代进行垃圾回收。如果希望对永久代进行垃圾回收，可用设置标志

**-XX:+DisableExplicitGC** 该标志将告诉JVM完全忽略系统的GC调用(system.gc())(不管使用的收集器是什么类型)

**-XX：+CMSConcurrentMTEnabled** 当该标志被启用时，并发的CMS阶段将以多线程执行

**-XX：ConcGCThreads** 比如value=4意味着CMS周期的所有阶段都以4个线程来执行

**-XX:SoftRefLRUPolicyMSPerMB=N** 如果SoftReference引用对象的生存时长<=空闲内存可保持软引用的最大时间范围，则不清除SoftReference所引用的对象；否则，则将其清除。soft reference 只会在垃圾回收时才会被清除，而垃圾回收并不总在发生。系统默认为一秒



**-XX:+UseG1GC** 开启g1收集器

**-XX:InitiatingHeapOccupancyPercent=45** 整个堆栈使用达到百分之多少的时候，启动GC周期. 基于整个堆，不仅仅是其中的某个代的占用情况，G1根据这个值来判断是否要触发GC周期, 0表示一直都在GC，默认值是45

**-XX:G1ReservePercent=n** 预留多少内存，防止晋升失败的情况，默认值是10

**-XX:NewRatio=n** 年轻代和老年代大小的比例，默认是2

**-XX:SurvivorRatio=n** eden和survivor区域空间大小的比例，默认是8

**-XX:MaxGCPauseMillis=n** 设置一个暂停时间期望目标，这是一个软目标，JVM会近可能的保证这个目标
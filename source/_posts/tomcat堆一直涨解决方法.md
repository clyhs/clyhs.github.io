---
title: tomcat堆一直涨解决方法
date: 2019-05-30 10:48:32
category: java
tags: tomcat
---

### loadrunner压测tomcat

在24小时压测过程中，发现tomcat的堆一直在涨，从2G到8G再到16G，压2小时过后，堆就满了，无法释放

#### 方法一：tomcat参数调优

tomcat的优化如下：

* (1)采用cms垃圾回收

catalina.sh

```shell
JAVA_OPTS="-server -showversion -Xms2g -Xmx16g -Xmn1g -XX:PermSize=256m -
JAVA_OPTS="$JAVA_OPTS -d64 -XX:CICompilerCount=8 -XX:+UseCompressedOops"
JAVA_OPTS="$JAVA_OPTS -XX:SurvivorRatio=4 -XX:TargetSurvivorRatio=90"
JAVA_OPTS="$JAVA_OPTS -XX:ReservedCodeCacheSize=256m -XX:-UseAdaptiveSizePolicy"
JAVA_OPTS="$JAVA_OPTS -Duser.timezone=Asia/Shanghai -XX:-DontCompileHugeMethods"
JAVA_OPTS="$JAVA_OPTS -Xss1024k -XX:+AggressiveOpts -XX:+UseBiasedLocking"
JAVA_OPTS="$JAVA_OPTS -XX:MaxTenuringThreshold=31 -XX:+CMSParallelRemarkEnabled "
JAVA_OPTS="$JAVA_OPTS -XX:GCTimeRatio=19 -XX:CMSFullGCsBeforeCompaction=0 -XX:+UseCMSCompactAtFullCollection  -XX:LargePageSizeInBytes=256m -XX:+UseFastAccessorMethods"
JAVA_OPTS="$JAVA_OPTS -XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=50 -Djava.awt.headless=true"
JAVA_OPTS="$JAVA_OPTS -XX:+UseGCOverheadLimit -XX:AllocatePrefetchDistance=256 -XX:AllocatePrefetchStyle=1"
JAVA_OPTS="$JAVA_OPTS -XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:MaxGCPauseMillis=200"
JAVA_OPTS="$JAVA_OPTS -XX:SoftRefLRUPolicyMSPerMB=0"

#JAVA_OPTS="$JAVA_OPTS -Dcom.sun.management.jmxremote.port=1090"
#JAVA_OPTS="$JAVA_OPTS -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false"   
#JAVA_OPTS="$JAVA_OPTS -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager" 
#JAVA_OPTS="$JAVA_OPTS -Djava.util.logging.config.file=$CATALINA_HOME\conf\logging.properties" 


```

server.xml

```shell
<Executor name="tomcatThreadPool" namePrefix="catalina-exec-" 
        maxThreads="800" maxIdleTime="60000" 
        minSpareThreads="100"/>
        
<Connector executor="tomcatThreadPool"   port="8170" protocol="org.apache.coyote.http11.Http11NioProtocol"
               connectionTimeout="20000"
               compression="on"  
               compressionMinSize="2048" 
               noCompressionUserAgents="gozilla, traviata"  
               compressableMimeType="text/html,application/xhtml+xml,application/xml,text/xml,text/javascript,text/css,text/plain,application/x-
javascript,application/javascript,text/xhtml,text/json,application/json,application/x-www-form-urlencoded,text/javaScript"  
               useSendfile="false"
               redirectPort="8443" />
```

* (2)采用G1回收

```shell
JAVA_OPTS="$JAVA_OPTS -XX:+UseG1GC -XX:InitiatingHeapOccupancyPercent=50 -XX:MaxGCPauseMillis=200"
JAVA_OPTS="$JAVA_OPTS -XX:ParallelGCThreads=8 -XX:ConcGCThreads=2 -XX:NewRatio=2 -XX:SurvivorRatio=8"
JAVA_OPTS="$JAVA_OPTS -XX:TargetSurvivorRatio=50 -XX:InitialTenuringThreshold=7 -XX:MaxTenuringThreshold=15"
JAVA_OPTS="$JAVA_OPTS -XX:G1ReservePercent=10 -XX:GCTimeRatio=19 -XX:+UnlockDiagnosticVMOptions"
```

*结果无语是cms还是g1都是2小时过后，堆就满了，可以定位系统内存泄漏，或者系统有问题，通过system.gc()方法强制回收是没有意义的，因为，用代码调用，只是通知jvm回收，但是jvm回不回收是自己决定，代码只起到通知作用而已。*

#### 方法二：分析堆内存

分析工具包括：jvisualvm.exe 和eclipse MAT等，eclipse MAT只需要在ecllipse装mat插件

在linux通过jps查看tomcat的pid，如：10045

导入堆文件命令：

```shell
jmap -dump:format=b,file=/home/heap.hprof  10045
```

图1：

![img](https://clyhs.github.io/images/tomcat/dump02.png)

图2：

![img](https://clyhs.github.io/images/tomcat/dump03.png)

图3：

![img](https://clyhs.github.io/images/tomcat/dump04.png)

可以看到org.crazycake.shiro.SessionInMemory

原来是session一直存放在内存，查了一下shiro-redis的源码，可以通过参数设置，使session不存放在内存，只存放到redis缓存

配置如下：

```xml
<bean id="redisSessionDAO" class="org.crazycake.shiro.RedisSessionDAO">
	    <property name="redisManager" ref="redisManager" />
		<property name="sessionInMemoryEnabled" value="false" />
	    <!-- optional properties
	    <property name="expire" value="-2"/>
	    <property name="keyPrefix" value="shiro:session:" />
	    -->
</bean>
```



再看下tomcat的监控：

如图：

![img](https://clyhs.github.io/images/tomcat/dump01.png)

再用loadrunner压测，堆始终不增长。










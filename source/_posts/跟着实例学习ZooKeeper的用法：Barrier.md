---
title: 跟着实例学习ZooKeeper的用法：Barrier
date: 2017-04-22 22:33:23
category: zookeeper
tags: [java,zookeeper]
---

[TOC]

分布式Barrier是这样一个类： 它会阻塞所有节点上的等待进程，知道某一个被满足， 然后所有的节点继续进行。

比如赛马比赛中， 等赛马陆续来到起跑线前。 一声令下，所有的赛马都飞奔而出。
# 栅栏Barrier
DistributedBarrier类实现了栅栏的功能。 它的构造函数如下：
```java
public DistributedBarrier(CuratorFramework client, String barrierPath)
Parameters:
client - client
barrierPath - path to use as the barrier
```
首先你需要设置栅栏，它将阻塞在它上面等待的线程:
```java
setBarrier();
```

然后需要阻塞的线程调用方法等待放行条件:
```java
public void waitOnBarrier()
```
当条件满足时，移除栅栏，所有等待的线程将继续执行：
```java
removeBarrier();
```
**异常处理**
DistributedBarrier 会监控连接状态，当连接断掉时waitOnBarrier()方法会抛出异常。

看一个例子：
```java
package com.colobu.zkrecipe.barrier;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.recipes.barriers.DistributedBarrier;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.apache.curator.test.TestingServer;
public class DistributedBarrierExample {
	private static final int QTY = 5;
	private static final String PATH = "/examples/barrier";
	public static void main(String[] args) throws Exception {
		try (TestingServer server = new TestingServer()) {
			CuratorFramework client = CuratorFrameworkFactory.newClient(server.getConnectString(), new ExponentialBackoffRetry(1000, 3));
			client.start();
			ExecutorService service = Executors.newFixedThreadPool(QTY);
			DistributedBarrier controlBarrier = new DistributedBarrier(client, PATH);
			controlBarrier.setBarrier();
			
			for (int i = 0; i < QTY; ++i) {
				final DistributedBarrier barrier = new DistributedBarrier(client, PATH);
				final int index = i;
				Callable<Void> task = new Callable<Void>() {
					@Override
					public Void call() throws Exception {
						
						Thread.sleep((long) (3 * Math.random()));
						System.out.println("Client #" + index + " waits on Barrier");
						barrier.waitOnBarrier();
						System.out.println("Client #" + index + " begins");
						return null;
					}
				};
				service.submit(task);
			}
			
			Thread.sleep(10000);
			System.out.println("all Barrier instances should wait the condition");
			
			
			controlBarrier.removeBarrier();
			
			
			service.shutdown();
			service.awaitTermination(10, TimeUnit.MINUTES);
		}
	}
}
```
这个例子创建了controlBarrier来设置栅栏和移除栅栏。
我们创建了5个线程，在此Barrier上等待。
最后移除栅栏后所有的线程才继续执行。

如果你开始不设置栅栏，所有的线程就不会阻塞住。
# 双栅栏Double Barrier
双栅栏允许客户端在计算的开始和结束时同步。当足够的进程加入到双栅栏时，进程开始计算， 当计算完成时，离开栅栏。
双栅栏类是DistributedDoubleBarrier。
构造函数为:
```java
public DistributedDoubleBarrier(CuratorFramework client,
                                String barrierPath,
                                int memberQty)
Creates the barrier abstraction. memberQty is the number of members in the barrier. When enter() is called, it blocks until
all members have entered. When leave() is called, it blocks until all members have left.
Parameters:
client - the client
barrierPath - path to use
memberQty - the number of members in the barrier
```

memberQty是成员数量，当enter方法被调用时，成员被阻塞，直到所有的成员都调用了enter。 当leave方法被调用时，它也阻塞调用线程， 知道所有的成员都调用了leave。
就像百米赛跑比赛， 发令枪响， 所有的运动员开始跑，等所有的运动员跑过终点线，比赛才结束。

DistributedBarrier 会监控连接状态，当连接断掉时enter()和leave方法会抛出异常。

例子代码：
```java
package com.colobu.zkrecipe.barrier;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.recipes.barriers.DistributedBarrier;
import org.apache.curator.framework.recipes.barriers.DistributedDoubleBarrier;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.apache.curator.test.TestingServer;
public class DistributedBarrierExample {
	private static final int QTY = 5;
	private static final String PATH = "/examples/barrier";
	public static void main(String[] args) throws Exception {
		try (TestingServer server = new TestingServer()) {
			CuratorFramework client = CuratorFrameworkFactory.newClient(server.getConnectString(), new ExponentialBackoffRetry(1000, 3));
			client.start();
			ExecutorService service = Executors.newFixedThreadPool(QTY);
			for (int i = 0; i < QTY; ++i) {
				final DistributedDoubleBarrier barrier = new DistributedDoubleBarrier(client, PATH, QTY);
				final int index = i;
				Callable<Void> task = new Callable<Void>() {
					@Override
					public Void call() throws Exception {
						
						Thread.sleep((long) (3 * Math.random()));
						System.out.println("Client #" + index + " enters");
						barrier.enter();
						System.out.println("Client #" + index + " begins");
						Thread.sleep((long) (3000 * Math.random()));
						barrier.leave();
						System.out.println("Client #" + index + " left");
						return null;
					}
				};
				service.submit(task);
			}
			
			
			service.shutdown();
			service.awaitTermination(10, TimeUnit.MINUTES);
		}
	}
}
```
分布式Barrier是这样一个类： 它会阻塞所有节点上的等待进程，知道某一个被满足， 然后所有的节点继续进行。

比如赛马比赛中， 等赛马陆续来到起跑线前。 一声令下，所有的赛马都飞奔而出。

---
title: 跟着实例学习zookeeper的用法：Leader选举
date: 2017-04-22 19:30:53
category: zookeeper
tags: [java,zookeeper]
---
# leader选举
在分布式计算中， leader election是很重要的一个功能， 这个选举过程是这样子的： 指派一个进程作为组织者，将任务分发给各节点。 在任务开始前， 哪个节点都不知道谁是leader或者coordinator. 当选举算法开始执行后， 每个节点最终会得到一个唯一的节点作为任务leader.
除此之外， 选举还经常会发生在leader意外宕机的情况下，新的leader要被选举出来。

Curator 有两种选举recipe， 你可以根据你的需求选择合适的。
## Leader latch
首先我们看一个使用`LeaderLatch`类来选举的例子。
它的构造函数如下：
```java
public LeaderLatch(CuratorFramework client, String latchPath)
public LeaderLatch(CuratorFramework client, String latchPath,  String id)
```
必须启动`LeaderLatch: leaderLatch.start()`;
一旦启动， `LeaderLatch`会和其它使用相同latch path的其它LeaderLatch交涉，然后随机的选择其中一个作为leader。 你可以随时查看一个给定的实例是否是leader:
```java
public boolean hasLeadership()
```
类似JDK的CountDownLatch， LeaderLatch在请求成为leadership时有block方法：
```java
public void await()
          throws InterruptedException,
                 EOFException
Causes the current thread to wait until this instance acquires leadership
unless the thread is interrupted or closed.
public boolean await(long timeout,
                     TimeUnit unit)
             throws InterruptedException
```
一旦不使用LeaderLatch了，必须调用close方法。 如果它是leader,会释放leadership， 其它的参与者将会选举一个leader。

**异常处理**
LeaderLatch实例可以增加ConnectionStateListener来监听网络连接问题。 当 SUSPENDED 或 LOST 时, leader不再认为自己还是leader.当LOST 连接重连后 RECONNECTED,LeaderLatch会删除先前的ZNode然后重新创建一个.
LeaderLatch用户必须考虑导致leadershi丢失的连接问题。 强烈推荐你使用ConnectionStateListener。

下面看例子：
```java
package com.colobu.zkrecipe.leaderelection;
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.List;
import java.util.concurrent.TimeUnit;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.recipes.leader.LeaderLatch;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.apache.curator.test.TestingServer;
import org.apache.curator.utils.CloseableUtils;
import com.google.common.collect.Lists;
public class LeaderLatchExample {
	private static final int CLIENT_QTY = 10;
	private static final String PATH = "/examples/leader";
	public static void main(String[] args) throws Exception {
		List<CuratorFramework> clients = Lists.newArrayList();
		List<LeaderLatch> examples = Lists.newArrayList();
		TestingServer server = new TestingServer();
		try {
			for (int i = 0; i < CLIENT_QTY; ++i) {
				CuratorFramework client = CuratorFrameworkFactory.newClient(server.getConnectString(), new ExponentialBackoffRetry(1000, 3));
				clients.add(client);
				LeaderLatch example = new LeaderLatch(client, PATH, "Client #" + i);
				examples.add(example);
				client.start();
				example.start();
			}
			Thread.sleep(20000);
			LeaderLatch currentLeader = null;
			for (int i = 0; i < CLIENT_QTY; ++i) {
				LeaderLatch example = examples.get(i);
				if (example.hasLeadership())
					currentLeader = example;
			}
			System.out.println("current leader is " + currentLeader.getId());
			System.out.println("release the leader " + currentLeader.getId());
			currentLeader.close();
			examples.get(0).await(2, TimeUnit.SECONDS);
			System.out.println("Client #0 maybe is elected as the leader or not although it want to be");
			System.out.println("the new leader is " + examples.get(0).getLeader().getId());
			
			System.out.println("Press enter/return to quit\n");
			new BufferedReader(new InputStreamReader(System.in)).readLine();
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			System.out.println("Shutting down...");
			for (LeaderLatch exampleClient : examples) {
				CloseableUtils.closeQuietly(exampleClient);
			}
			for (CuratorFramework client : clients) {
				CloseableUtils.closeQuietly(client);
			}
			CloseableUtils.closeQuietly(server);
		}
	}
}
```
首先我们创建了10个LeaderLatch，启动后它们中的一个会被选举为leader。 因为选举会花费一些时间，start后并不能马上就得到leader。
通过hasLeadership查看自己是否是leader， 如果是的话返回true。
可以通过.getLeader().getId()可以得到当前的leader的ID。
只能通过close释放当前的领导权。
await是一个阻塞方法， 尝试获取leader地位，但是未必能上位。
##Leader Election
Curator还提供了另外一种选举方法。
注意涉及以下四个类：

    LeaderSelector
    LeaderSelectorListener
    LeaderSelectorListenerAdapter
    CancelLeadershipException

重要的是LeaderSelector类，它的构造函数为：
```java
public LeaderSelector(CuratorFramework client, String mutexPath,LeaderSelectorListener listener)
public LeaderSelector(CuratorFramework client, String mutexPath, ThreadFactory threadFactory, Executor executor, LeaderSelectorListener listener)
```
类似LeaderLatch,必须start: leaderSelector.start();
一旦启动，当实例取得领导权时你的listener的takeLeadership()方法被调用. 而takeLeadership()方法只有领导权被释放时才返回。
当你不再使用LeaderSelector实例时，应该调用它的close方法。

**异常处理**
LeaderSelectorListener类继承ConnectionStateListener.LeaderSelector必须小心连接状态的改变. 如果实例成为leader, 它应该相应SUSPENDED 或 LOST. 当 SUSPENDED 状态出现时， 实例必须假定在重新连接成功之前它可能不再是leader了。 如果LOST状态出现， 实例不再是leader， takeLeadership方法返回.

重要: 推荐处理方式是当收到SUSPENDED 或 LOST时抛出CancelLeadershipException异常. 这会导致LeaderSelector实例中断并取消执行takeLeadership方法的异常. 这非常重要， 你必须考虑扩展LeaderSelectorListenerAdapter. LeaderSelectorListenerAdapter提供了推荐的处理逻辑。

这个例子摘自官方。
首先创建一个ExampleClient类， 它继承LeaderSelectorListenerAdapter， 它实现了takeLeadership方法：
```java
package com.colobu.zkrecipe.leaderelection;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.recipes.leader.LeaderSelectorListenerAdapter;
import org.apache.curator.framework.recipes.leader.LeaderSelector;
import java.io.Closeable;
import java.io.IOException;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;
public class ExampleClient extends LeaderSelectorListenerAdapter implements Closeable {
	private final String name;
	private final LeaderSelector leaderSelector;
	private final AtomicInteger leaderCount = new AtomicInteger();
	public ExampleClient(CuratorFramework client, String path, String name) {
		this.name = name;
		leaderSelector = new LeaderSelector(client, path, this);
		leaderSelector.autoRequeue();
	}
	public void start() throws IOException {
		leaderSelector.start();
	}
	@Override
	public void close() throws IOException {
		leaderSelector.close();
	}
	
	@Override
	public void takeLeadership(CuratorFramework client) throws Exception {
		final int waitSeconds = (int) (5 * Math.random()) + 1;
		System.out.println(name + " is now the leader. Waiting " + waitSeconds + " seconds...");
		System.out.println(name + " has been leader " + leaderCount.getAndIncrement() + " time(s) before.");
		try {
			Thread.sleep(TimeUnit.SECONDS.toMillis(waitSeconds));
		} catch (InterruptedException e) {
			System.err.println(name + " was interrupted.");
			Thread.currentThread().interrupt();
		} finally {
			System.out.println(name + " relinquishing leadership.\n");
		}
	}
}
```
你可以在takeLeadership进行任务的分配等等，并且不要返回，如果你想要要此实例一直是leader的话可以加一个死循环。
leaderSelector.autoRequeue();保证在此实例释放领导权之后还可能获得领导权。
在这里我们使用AtomicInteger来记录此client获得领导权的次数， 它是"fair"， 每个client有平等的机会获得领导权。

测试代码:
```java
package com.colobu.zkrecipe.leaderelection;
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.util.List;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.recipes.leader.LeaderSelector;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.apache.curator.test.TestingServer;
import org.apache.curator.utils.CloseableUtils;
import com.google.common.collect.Lists;
public class LeaderSelectorExample {
	private static final int CLIENT_QTY = 10;
	private static final String PATH = "/examples/leader";
	public static void main(String[] args) throws Exception {
		List<CuratorFramework> clients = Lists.newArrayList();
		List<ExampleClient> examples = Lists.newArrayList();
		TestingServer server = new TestingServer();
		try {
			for (int i = 0; i < CLIENT_QTY; ++i) {
				CuratorFramework client = CuratorFrameworkFactory.newClient(server.getConnectString(), new ExponentialBackoffRetry(1000, 3));
				clients.add(client);
				ExampleClient example = new ExampleClient(client, PATH, "Client #" + i);
				examples.add(example);
				client.start();
				example.start();
			}
			
			System.out.println("Press enter/return to quit\n");
			new BufferedReader(new InputStreamReader(System.in)).readLine();
		} finally {
			System.out.println("Shutting down...");
			for (ExampleClient exampleClient : examples) {
				CloseableUtils.closeQuietly(exampleClient);
			}
			for (CuratorFramework client : clients) {
				CloseableUtils.closeQuietly(client);
			}
			CloseableUtils.closeQuietly(server);
		}
	}
}
```
与LeaderLatch， 通过LeaderSelectorListener可以对领导权进行控制， 在适当的时候释放领导权，这样每个节点都有可能获得领导权。 而LeaderLatch一根筋到死， 除非调用close方法，否则它不会释放领导权。
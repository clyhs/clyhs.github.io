---
title: 跟着实例学习ZooKeeper的用法： 队列
date: 2017-04-22 22:49:47
category: zookeeper
tags: [java,zookeeper]
---
Curator也提供ZK Recipe的分布式队列实现。 利用ZK的 PERSISTENTSEQUENTIAL节点， 可以保证放入到队列中的项目是按照顺序排队的。 如果单一的消费者从队列中取数据， 那么它是先入先出的，这也是队列的特点。 如果你严格要求顺序，你就的使用单一的消费者，可以使用leader选举只让leader作为唯一的消费者。

但是， 根据Netflix的Curator作者所说， ZooKeeper真心不适合做Queue，或者说ZK没有实现一个好的Queue，详细内容可以看 Tech Note 4， 原因有五：

    1.  ZK有1MB 的传输限制。 实践中ZNode必须相对较小，而队列包含成千上万的消息，非常的大。
    2. 如果有很多节点，ZK启动时相当的慢。 而使用queue会导致好多ZNode. 你需要显著增大 initLimit 和 syncLimit.
    3. ZNode很大的时候很难清理。Netflix不得不创建了一个专门的程序做这事。
    4. 当很大量的包含成千上万的子节点的ZNode时， ZK的性能变得不好
    5. ZK的数据库完全放在内存中。 大量的Queue意味着会占用很多的内存空间。

尽管如此， Curator还是创建了各种Queue的实现。 如果Queue的数据量不太多，数据量不太大的情况下，酌情考虑，还是可以使用的。
# DistributedQueue
DistributedQueue是最普通的一种队列。 它设计以下四个类：

    QueueBuilder
    QueueConsumer
    QueueSerializer
    DistributedQueue

创建队列使用QueueBuilder,它也是其它队列的创建类。
```java
public static <T> QueueBuilder<T> builder(CuratorFramework client,
                                          QueueConsumer<T> consumer,
                                          QueueSerializer<T> serializer,
                                          java.lang.String queuePath)
```
```
QueueBuilder<MessageType>    builder = QueueBuilder.builder(client, consumer, serializer, path);
... more builder method calls as needed ...
DistributedQueue<MessageType queue = builder.build();	`
```
创建好queue就可以往里面放入数据了：
```java
queue.put(aMessage);
```
QueueSerializer提供了对队列中的对象的序列化和反序列化。

QueueConsumer是消费者， 它可以接收队列的数据。 处理队列中的数据的代码逻辑可以放在QueueConsumer.consumeMessage()中。

正常情况下先将消息从队列中移除，再交给消费者消费。 但这是两个步骤，不是原子的。 可以调用Builder的lockPath()消费者加锁， 当消费者消费数据时持有锁，这样其它消费者不能消费此消息。 如果消费失败或者进程死掉，消息可以交给其它进程。这会带来一点性能的损失。 最好还是单消费者模式使用队列。

例子：
```java
package com.colobu.zkrecipe.queue;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.api.CuratorEvent;
import org.apache.curator.framework.api.CuratorListener;
import org.apache.curator.framework.recipes.queue.DistributedQueue;
import org.apache.curator.framework.recipes.queue.QueueBuilder;
import org.apache.curator.framework.recipes.queue.QueueConsumer;
import org.apache.curator.framework.recipes.queue.QueueSerializer;
import org.apache.curator.framework.state.ConnectionState;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.apache.curator.test.TestingServer;
import org.apache.curator.utils.CloseableUtils;
public class DistributedQueueExample {
	private static final String PATH = "/example/queue";
	public static void main(String[] args) throws Exception {
		TestingServer server = new TestingServer();
		CuratorFramework client = null;
		DistributedQueue<String> queue = null;
		try {
			client = CuratorFrameworkFactory.newClient(server.getConnectString(), new ExponentialBackoffRetry(1000, 3));
			client.getCuratorListenable().addListener(new CuratorListener() {
				@Override
				public void eventReceived(CuratorFramework client, CuratorEvent event) throws Exception {
					System.out.println("CuratorEvent: " + event.getType().name());
				}
			});
			client.start();
			QueueConsumer<String> consumer = createQueueConsumer();
			QueueBuilder<String> builder = QueueBuilder.builder(client, consumer, createQueueSerializer(), PATH);
			queue = builder.buildQueue();
			queue.start();
			
			for (int i = 0; i < 10; i++) {
				queue.put(" test-" + i);
				Thread.sleep((long)(3 * Math.random()));
			}
			
			Thread.sleep(20000);
			
		} catch (Exception ex) {
		} finally {
			CloseableUtils.closeQuietly(queue);
			CloseableUtils.closeQuietly(client);
			CloseableUtils.closeQuietly(server);
		}
	}
	private static QueueSerializer<String> createQueueSerializer() {
		return new QueueSerializer<String>(){
			@Override
			public byte[] serialize(String item) {
				return item.getBytes();
			}
			@Override
			public String deserialize(byte[] bytes) {
				return new String(bytes);
			}
			
		};
	}
	private static QueueConsumer<String> createQueueConsumer() {
		return new QueueConsumer<String>(){
			@Override
			public void stateChanged(CuratorFramework client, ConnectionState newState) {
				System.out.println("connection new state: " + newState.name());
			}
			@Override
			public void consumeMessage(String message) throws Exception {
				System.out.println("consume one message: " + message);				
			}
			
		};
	}
}
```
# DistributedIdQueue
# DistributedPriorityQueue
# DistributedDelayQueue
# SimpleDistributedQueue
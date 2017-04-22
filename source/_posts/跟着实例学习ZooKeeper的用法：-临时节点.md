---
title: 跟着实例学习ZooKeeper的用法： 临时节点
date: 2017-04-22 22:47:53
category: zookeeper
tags: [java,zookeeper]
---
使用Curator也可以简化Ephemeral Node (临时节点)的操作。
临时节点驻存在ZooKeeper中，当连接和session断掉时被删除。

比如通过ZooKeeper发布服务，服务启动时将自己的信息注册为临时节点，当服务断掉时ZooKeeper将此临时节点删除，这样client就不会得到服务的信息了。

PersistentEphemeralNode类代表临时节点。
通过下面的构造函数创建：
```java
public PersistentEphemeralNode(CuratorFramework client,
                               PersistentEphemeralNode.Mode mode,
                               String basePath,
                               byte[] data)
Parameters:
client - client instance
mode - creation/protection mode
basePath - the base path for the node
data - data for the node
```
其它参数还好理解， 不好理解的是PersistentEphemeralNode.Mode。

    EPHEMERAL： 以ZooKeeper的 CreateMode.EPHEMERAL方式创建节点。
    EPHEMERAL_SEQUENTIAL: 如果path已经存在，以CreateMode.EPHEMERAL创建节点，否则以CreateMode.EPHEMERAL_SEQUENTIAL方式创建节点。
    PROTECTED_EPHEMERAL: 以CreateMode.EPHEMERAL创建，提供保护方式。
    PROTECTED_EPHEMERAL_SEQUENTIAL: 类似EPHEMERAL_SEQUENTIAL，提供保护方式。

保护方式是指一种很边缘的情况： 当服务器将节点创建好，但是节点名还没有返回给client,这时候服务器可能崩溃了，然后此时ZK session仍然合法， 所以此临时节点不会被删除。对于client来说， 它无法知道哪个节点是它们创建的。

即使不是sequential-ephemeral,也可能服务器创建成功但是客户端由于某些原因不知道创建的节点。

Curator对这些可能无人看管的节点提供了保护机制。 这些节点创建时会加上一个GUID。 如果节点创建失败正常的重试机制会发生。 重试时， 首先搜索父path, 根据GUID搜索节点，如果找到这样的节点， 则认为这些节点是第一次尝试创建时创建成功但丢失的节点，然后返回给调用者。

节点必须调用start方法启动。 不用时调用close方法。

PersistentEphemeralNode 内部自己处理错误状态。

我们的例子创建了两个节点，一个是临时节点，一个事持久化的节点。 可以看到， client重连后临时节点不存在了。
```java
package com.colobu.zkrecipe.node;
import java.util.concurrent.TimeUnit;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.recipes.nodes.PersistentEphemeralNode;
import org.apache.curator.framework.recipes.nodes.PersistentEphemeralNode.Mode;
import org.apache.curator.framework.state.ConnectionState;
import org.apache.curator.framework.state.ConnectionStateListener;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.apache.curator.test.KillSession;
import org.apache.curator.test.TestingServer;
import org.apache.curator.utils.CloseableUtils;
public class PersistentEphemeralNodeExample {
	private static final String PATH = "/example/ephemeralNode";
	private static final String PATH2 = "/example/node";
	
	public static void main(String[] args) throws Exception {
		TestingServer server = new TestingServer();
		CuratorFramework client = null;
		PersistentEphemeralNode node = null;
		try {
			client = CuratorFrameworkFactory.newClient(server.getConnectString(), new ExponentialBackoffRetry(1000, 3));
			client.getConnectionStateListenable().addListener(new ConnectionStateListener() {
				
				@Override
				public void stateChanged(CuratorFramework client, ConnectionState newState) {
					System.out.println("client state:" + newState.name());
					
				}
			});
			client.start();
			
			//http://zookeeper.apache.org/doc/r3.2.2/api/org/apache/zookeeper/CreateMode.html
			node = new PersistentEphemeralNode(client, Mode.EPHEMERAL,PATH, "test".getBytes());
			node.start();
			node.waitForInitialCreate(3, TimeUnit.SECONDS);
			String actualPath = node.getActualPath();
			System.out.println("node " + actualPath + " value: " + new String(client.getData().forPath(actualPath)));
			
			client.create().forPath(PATH2, "persistent node".getBytes());
			System.out.println("node " + PATH2 + " value: " + new String(client.getData().forPath(PATH2)));
			KillSession.kill(client.getZookeeperClient().getZooKeeper(), server.getConnectString());
			System.out.println("node " + actualPath + " doesn't exist: " + (client.checkExists().forPath(actualPath) == null));
			System.out.println("node " + PATH2 + " value: " + new String(client.getData().forPath(PATH2)));
			
		} catch (Exception ex) {
			ex.printStackTrace();
		} finally {
			CloseableUtils.closeQuietly(node);
			CloseableUtils.closeQuietly(client);
			CloseableUtils.closeQuietly(server);
		}
	}
}
```